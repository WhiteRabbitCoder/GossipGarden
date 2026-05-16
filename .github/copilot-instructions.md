# Copilot instructions for the GossipGarden workspace

This workspace contains **two independent projects**:

- `backendGossipGarden/` (FastAPI backend, Python 3.11+)
- `frontendGossipGarden/src/` (Flutter app)

Use project-local docs as primary context:

- Backend: `backendGossipGarden/API_CONTRACT.md`, `backendGossipGarden/.github/copilot-instructions.md`, `backendGossipGarden/.github/instructions/*.md.instructions.md`
- Frontend: `frontendGossipGarden/src/README.md`, `frontendGossipGarden/API_CONTRACT.md`, `frontendGossipGarden/CLAUDE.md`

## Build, test, and lint commands

### Backend (`backendGossipGarden/`)

| Task | Command |
| --- | --- |
| Install deps | `cd backendGossipGarden && python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` |
| Run API (dev) | `cd backendGossipGarden && uvicorn app.main:app --reload --host 0.0.0.0 --port 8000` |
| Run with Docker | `cd backendGossipGarden && docker compose up --build` |
| MQTT publish smoke script | `cd backendGossipGarden && python test_mqtt_publish.py` |

Backend automated test/lint commands are **not currently defined in repo config** (no `tests/` suite, `pytest.ini`, or linter config files committed).

### Frontend (`frontendGossipGarden/src/`)

| Task | Command |
| --- | --- |
| Install deps | `cd frontendGossipGarden/src && flutter pub get` |
| Lint | `cd frontendGossipGarden/src && flutter analyze` |
| Run tests | `cd frontendGossipGarden/src && flutter test` |
| Run a single test file | `cd frontendGossipGarden/src && flutter test test/widget_test.dart` |
| Run one test by name | `cd frontendGossipGarden/src && flutter test --plain-name "App renders login entry point"` |
| Build APK | `cd frontendGossipGarden/src && flutter build apk --dart-define=BACKEND_TARGET=remote` |
| Run against local backend (Android emulator) | `cd frontendGossipGarden/src && flutter run --dart-define=BACKEND_TARGET=local --dart-define=BACKEND_LOCAL_URL=http://10.0.2.2:8000` |

## High-level architecture

### Backend (`backendGossipGarden`)

- `app/main.py` boots FastAPI, checks Supabase/Firebase/Redis during lifespan, and mounts `/api/v1` routes from `app/api/v1/api.py`.
- Routers currently mounted: `auth`, `plants`, `sensors`, `identification`.
- Persistence is polyglot:
  - **Supabase/Postgres**: relational entities (plants, species + 4 child tables, users, friendships, etc.).
  - **Firestore**: telemetry documents under `plants/{plant_id}/sensor_readings`.
  - **Redis**: async cache/client via `redis.asyncio`.
  - **pgvector (Supabase)**: `botanical_chunks` table with `text-embedding-3-small` (1536d) embeddings for RAG.
- MQTT ingestion (`app/core/mqtt.py`) is optional (`MQTT_ENABLED`), subscribes to `plantas/+/sensores`, and writes 30-day TTL telemetry docs (`expireAt`) into Firestore.
- Plant identification pipeline (`app/api/v1/endpoints/identification.py`): `POST /api/v1/identify` accepts multipart image, decides by Plant.id confidence threshold (needs_more_photos / needs_user_selection / completed). Full pipeline: Plant.id → GBIF → pgvector RAG → OpenAI gpt-4o Structured Output → Supabase. `POST /api/v1/species/from-candidate` runs the full pipeline for a user-selected candidate. Results cached by `scientific_name`. Schema migration documented in `backendGossipGarden/docs/species-schema-migration.md`.

### Frontend (`frontendGossipGarden/src`)

- App entry (`lib/main.dart`) initializes Firebase conditionally, then routes via `_AppGate` (login/onboarding/main).
- State is Riverpod-driven (`auth_provider`, `navigation_provider`, `chat_providers`, `plant_providers`).
- Navigation is a **single `Scaffold` + `IndexedStack` + overlay panels** (`MainScreen`), not route-push-driven.
- Chat supports dual persistence paths:
  - Firestore (`plants/{plantId}/messages`) when Firebase is configured.
  - In-memory repository fallback when it is not.
- Plant REST/SSE mapping logic lives in `PlantApiDatasource` and `SensorStreamDatasource` (field alias handling, status derivation, health/mood/insight calculations).

## Key conventions for this codebase

1. **Do not assume schema or field names.** Follow `API_CONTRACT.md` and schema docs; this repo contains mixed legacy/current contracts.
2. **Backend config must flow through `app/core/config.py` (`pydantic-settings`)**, not ad-hoc `os.getenv` reads in feature files.
3. **Backend auth is Supabase JWT via JWKS (ES256 + audience `authenticated`)** in `app/core/security.py`; do not switch to HS256/shared-secret validation.
4. **Backend DB clients are singleton-style modules** (`app/db/supabase.py`, `firebase.py`, `redis.py`) and are imported/reused directly.
5. **MQTT payloads must include `plant_id`**; messages without it are intentionally ignored.
6. **Frontend runtime config is compile-time (`--dart-define`)**, not `.env` files; backend target resolution is in `lib/core/config/app_config.dart`.
7. **Firebase is optional by design**: auth/chat must keep working in fallback mode when Firebase keys are not provided.
8. **`plant_providers.dart` is currently demo-oriented** (mock plant list + synthetic realtime stream). Real HTTP/SSE data sources exist but are not the default provider wiring.
9. **UI strings are predominantly Spanish**; keep new user-facing text consistent with surrounding language in each file.
10. **Chat backend URL is hardcoded in `BackendChatService`** (separate from `AppConfig`), so backend target changes in `--dart-define` do not affect plant-chat calls unless that service is updated too.
