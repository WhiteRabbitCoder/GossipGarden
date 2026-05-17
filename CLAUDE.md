# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

This workspace contains two independent sub-projects, each with its own git repo:

- `backendGossipGarden/` — FastAPI (Python 3.11+) REST API. Active development here.
- `frontendGossipGarden/` — Flutter mobile app (`src/` inside). See `frontendGossipGarden/CLAUDE.md` for Flutter-specific guidance.

The two projects communicate: the Flutter app consumes the FastAPI backend. `backendGossipGarden/API_CONTRACT.md` is the canonical spec for that interface.

---

## backendGossipGarden

### Setup

```bash
cd backendGossipGarden
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # fill in SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, SUPABASE_JWKS_URL,
                       # FIREBASE_CREDENTIALS_PATH, FIREBASE_STORAGE_BUCKET, REDIS_URL,
                       # PLANT_ID_API_KEY, OPENAI_API_KEY
```

### Running

```bash
# Dev (auto-reload)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Or via Docker (includes Redis)
docker compose up
```

Swagger UI at `http://localhost:8000/docs`.

Production deployment: `https://backendgossipgarden-production.up.railway.app`

### Tests

```bash
pytest                          # all unit tests (mocked externals)
pytest tests/test_chat.py       # single file
pytest -k "test_health_ok"      # single test by name
```

`pytest.ini` sets `asyncio_mode = auto`, `testpaths = tests`, and defines custom marker `dbschema` for tests requiring a live Postgres with pgvector. Integration tests (class `TestIntegracion`) hit a real server at `:8000` — they auto-skip if the backend isn't up.

### CI (GitHub Actions)

**Backend** (`.github/workflows/ci.yml`): `uv` for deps, Python 3.12, two jobs:
1. **Unit tests** — `pytest -m "not dbschema"` with mocked externals + coverage.
2. **DB schema validation** — ephemeral `pgvector/pgvector:pg16`; applies `schema.sql` from scratch and `migrations.sql` over legacy schema; runs `pytest -m dbschema`.

**Frontend** (`.github/workflows/ci.yml`): Flutter 3.x stable, two jobs:
1. **Analyze** — `flutter analyze --fatal-infos`.
2. **Test** — `flutter test --coverage` with 40% threshold on business logic (excludes screens/widgets/main).

---

### Architecture

**Polyglot Persistence** — four stores with distinct roles:

| Store | SDK | Role |
|---|---|---|
| Supabase (PostgreSQL) | `supabase-py` + `SUPABASE_SERVICE_ROLE_KEY` | Relational data: users, plants, species (+ 3 child tables), sensors, events, friendships, monthly_metrics |
| Firebase Firestore | `firebase-admin` | IoT telemetry (`sensor_readings` with 30-day TTL), chat logs, and `plant_identifications` metadata |
| Firebase Storage | `firebase-admin` (`storage`) | Plant photos (compressed JPEG, max 1920px) — paths stored in `plants.photo_storage_path` |
| Redis (async) | `redis` | Chat history cache (key `chat:{user_id}:{plant_id}`, 2h TTL) + summary cache (key `chat:summary:{user_id}:{plant_id}`, 7d TTL) |
| pgvector (en Supabase) | `supabase-py` RPC | `botanical_chunks` con embeddings `text-embedding-3-small` (1536d) para RAG |

**Request flow:**
1. `app/main.py` — FastAPI app with lifespan that validates all three DB connections and starts MQTT on startup.
2. `app/api/v1/api.py` — router that mounts `auth`, `plants`, `sensors`, `identification`, `chat` at `/api/v1`.
3. `app/core/security.py` — `get_current_user` dependency: validates Supabase-issued JWTs via JWKS (ES256, audience `"authenticated"`), returns `user_id` UUID string.
4. `app/core/config.py` — `pydantic-settings` `Settings` object; all config comes from `.env`.
5. **Plant identification** (`app/api/v1/endpoints/identification.py`): `POST /api/v1/identify` receives multipart image + optional lat/lon, decides by confidence threshold (plant.id probability): `needs_more_photos` (<0.25) / `needs_user_selection` (0.25–0.75, top 3) / `completed` (>0.75, runs full pipeline). `POST /api/v1/species/from-candidate` completes the pipeline when the user selects from top-3. Full pipeline: plant.id → GBIF → RAG (pgvector) → OpenAI gpt-4o Structured Output → Supabase. Cached by `scientific_name` to avoid repeat paid API calls. On `completed`, the image is compressed (Pillow, max 1920px, JPEG q=85) and uploaded to Firebase Storage via `BackgroundTasks` (zero latency impact); `photo_storage_path` is returned in the response so the client can link the photo when creating the plant. `PUT /api/v1/plants/{plant_id}/photo` allows updating a plant's photo without re-identifying.

**LLM chat** (`app/api/v1/endpoints/chat.py` + `app/services/chat_service.py`):
- Calls **OpenAI** via `AsyncOpenAI` with model `OPENAI_CHAT_MODEL` (default `gpt-4o`).
- Two-level memory:
  - **Redis**: short-term cache — history (2h TTL), summaries (7d TTL).
  - **Firestore**: long-term permanent storage — `plants/{plant_id}/chat_logs/{user_id}` (messages), `plants/{plant_id}/chat_meta/{user_id}` (compacted summaries).
- Context compaction (`app/services/summarizer_service.py`): when history exceeds 3000 tokens, older messages are summarized by GPT and the summary is injected into the system prompt. Last 6 messages (3 turns) always preserved intact.
- Guardrails: strict persona enforcement — the plant cannot discuss politics, programming, or off-topic subjects; always redirects to plant-related topics.
- Injects latest sensor data (from Firestore) into the system prompt as real-time plant status.
- Last 10 turn pairs max in active history.
- Endpoints:
  - `POST /api/v1/chat/{plant_id}` — blocking chat.
  - `GET /api/v1/chat/{plant_id}/history` — conversation history.

**MQTT** (`app/core/mqtt.py`):
- Disabled by default (`MQTT_ENABLED=false`). When enabled, subscribes to `plantas/+/sensores`.
- Messages must include `plant_id` in the JSON payload; writes a `sensor_readings` doc to Firestore under `plants/{plant_id}/sensor_readings`.

**Health scoring** (`app/services/health_service.py`):
- Called on sensor data ingestion. Fetches `species_care_profiles` for the plant's species.
- Calculates weighted parameter scores (temperature, light, air humidity, soil humidity) using the care profile's `weight_*` fields (defaults to equal 0.25 each).
- Status thresholds: >=80 → "healthy", >=50 → "warning", <50 → "critical".
- Updates `plants.health_score` and `plants.health_status` in Supabase.

**Firebase init** (`app/db/firebase.py`): if `FIREBASE_CREDENTIALS_JSON` env var is set (Railway/PaaS), it parses the JSON inline; otherwise falls back to `FIREBASE_CREDENTIALS_PATH` file. If `FIREBASE_STORAGE_BUCKET` is set, Storage is initialized at the same time (format: `project-id.appspot.com` or `project-id.firebasestorage.app`, no `gs://` prefix). Missing credentials log a warning and disable Firestore gracefully.

**Image storage** (`app/services/image_storage_service.py`): `compress_image()` resizes to max 1920px and re-encodes as JPEG q=85 using Pillow. `compute_storage_path()` generates a deterministic path before the background task so the endpoint can include it in the response. `store_identification_result()` runs as a `BackgroundTask` after `/identify` responds. `upload_plant_photo()` is called synchronously by `PUT /plants/{plant_id}/photo`.

---

### PostgreSQL schema (Supabase) — do not assume, always follow this

The canonical SQL lives in `backendGossipGarden/migrations/`:

- **`schema.sql`** — script completo para crear la BD desde cero (entornos nuevos / CI).
- **`migrations.sql`** — migraciones incrementales unificadas (001 + 002 + 003) para aplicar sobre una BD legacy existente.

Resumen de tablas:

```
users:                   user_id (UUID PK), username, email, created_at
species:                 id (UUID PK), scientific_name (TEXT UNIQUE), common_name,
                         family, genus, gbif_taxon_key (int), inaturalist_id (int),
                         source_provider, created_at, updated_at
                         ⚠ NO care ranges, NO ai_personality_prompt — see child tables
species_care_profiles:   id (UUID PK), species_id (FK), min/max_temp_c, min/max_light_lux,
                         min/max_air_humidity_pct, min/max_soil_humidity_pct,
                         care_data_source, proposal_confidence, needs_review (bool),
                         reasoning_summary, completed_at,
                         weight_light/weight_soil_humidity/weight_air_humidity/weight_temperature (FLOAT nullable, 0–1, suma≈1.0),
                         sensitivity_light/sensitivity_soil_humidity/sensitivity_air_humidity/sensitivity_temperature (TEXT nullable: 'high'|'medium'|'low')
species_ai_content:      id (UUID PK), species_id (FK), ai_personality_prompt, care_summary,
                         care_tips (JSONB), fun_facts (JSONB), faq (JSONB),
                         language, llm_model, generated_at  — UNIQUE (species_id, language)
species_common_names:    id (UUID PK), species_id (FK), name, language, region
                         — UNIQUE (species_id, name, language)
botanical_chunks:        id (UUID PK), content, embedding (vector(1536)), source,
                         scientific_name, family, metadata (JSONB), created_at
                         — RPC: match_botanical_chunks(query_embedding, match_count,
                                min_similarity, family_filter, scientific_filter)
plants:                  plant_id (UUID PK), user_id (FK), species_id (FK → species.id),
                         nickname, health_status (enum: healthy/warning/critical),
                         health_score (float), photo_storage_path (TEXT nullable),
                         last_health_check, created_at
sensors:                 sensor_id (UUID PK), plant_id (UUID FK nullable), mac_address, is_online, last_ping
events:                  event_id (UUID PK), plant_id (FK), type (enum: alert/insight/chat/system), message, created_at
friendships:             id (UUID PK), user_low_id (FK), user_high_id (FK), requested_by_id (FK),
                         status (enum: pending/accepted/blocked), created_at
monthly_metrics:         id (UUID PK), plant_id (FK), month, year, avg_temperature,
                         avg_soil_humidity, avg_air_humidity, avg_light,
                         avg_health_score (float), health_status_majority (varchar)
species_legacy:          backup post-migration — DROP after 30-day validation period
```

---

## frontendGossipGarden (Flutter)

See `frontendGossipGarden/CLAUDE.md` for full details. Quick reference:

```bash
cd frontendGossipGarden/src
flutter pub get
flutter analyze
flutter test
flutter run --dart-define=BACKEND_TARGET=local   # → http://10.0.2.2:8000 (Android emulator)
flutter run --dart-define=BACKEND_TARGET=prod    # → Railway production
```
