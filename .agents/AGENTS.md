# Workspace Customization Rules

## Integración de Ramas y Lógica
Cuando se te pida traer lógica o componentes de otras ramas o entornos:
1. **NO sobrescribas los archivos en bruto (raw)** usando `git checkout` o reemplazos masivos sin previo análisis.
2. **Lee y comprende** primero la lógica y el diseño del archivo actual y del archivo entrante.
3. **Analiza las diferencias** para identificar qué tienen en común y qué cambia.
4. **Implementa gradualmente** las diferencias, asegurándote de mantener el estilo, diseño e interfaz original de la rama destino, e integrando solo la lógica subyacente solicitada (dando consideración especial a las partes más importantes).

---

# AGENTS.md - Backend Guide

This document is designed to provide context and guidance for any AI agent or developer interacting with the `backendGossipGarden` repository. It provides an analysis of the structure, the general context of its functions, and rules for extending, refactoring, or migrating code.

## 1. Architectural Structure

The backend is a **FastAPI** application relying on a polyglot persistence strategy:
- **Supabase (PostgreSQL)**: Primary relational store and pgvector (RAG) backend.
- **Firebase Firestore**: Telemetry, chat logs, JSON metadata.
- **Firebase Storage**: Image hosting.
- **Redis**: Fast, short-term and medium-term caching.

### Directory Breakdown
- **`app/main.py`**: The main entry point. Handles the app lifecycle (`lifespan`), validating DB connections, and starting MQTT if enabled.
- **`app/api/v1/`**: The HTTP routing layer. Contains endpoints grouped by domain (`plants`, `auth`, `sensors`, `chat`, `identification`). Handlers here should be kept thin.
- **`app/services/`**: The core business logic layer. All heavy lifting, third-party API interactions (e.g., OpenAI, plant.id, GBIF), and complex database operations reside here.
- **`app/schemas/`**: Pydantic v2 data models for request validation and response formatting.
- **`app/core/`**: Core utilities, including configuration (`config.py`), security (`security.py`), and background tasks like MQTT (`mqtt.py`).
- **`app/db/`**: Database client initializations (Singletons). Never import these directly into endpoints; use dependency injection (`Depends`).

## 2. General Context of Key Functions

- **Plant Identification (`app/api/v1/endpoints/identification.py` & Services)**: 
  Handles multipart image uploads. Calls `plant.id` -> `GBIF` taxonomy -> `pgvector` RAG -> `OpenAI` to determine species. Uses confidence thresholds to determine if more user input is needed. Triggers background tasks to store images in Firebase.
- **Chatbot / LLM Pipeline (`app/api/v1/endpoints/chat.py` & `chat_service.py`)**: 
  Injects plant personality and sensor context into the system prompt. Uses Redis for active short-term context and Firestore for permanent logging. Uses a summarizer service when the context window gets too large.
- **Health Scoring (`health_service.py`)**:
  Calculates a weighted health score based on real-time sensor data and species care profiles. Updates Supabase records with `healthy`, `warning`, or `critical`.
- **Image Storage (`image_storage_service.py`)**:
  Compresses images using Pillow and coordinates deterministic path generation and Firebase uploads synchronously or as background tasks.
- **Security (`core/security.py`)**:
  Validates Supabase-issued JWTs. Every protected route depends on this.

## 3. Preparation for Migrations, Refactoring, and New Features

When refactoring or adding functionalities, strictly adhere to these rules:

1. **Keep Endpoints Thin**: Move all domain logic, API calls, and DB querying to `app/services/`. The API layer should only handle validation (via schemas) and HTTP responses.
2. **Dependency Injection**: Use FastAPI `Depends()` for DB and Auth. This simplifies unit testing.
3. **Database Schema is King**: Never assume database column names. Always check `migrations/schema.sql`. For instance, `plants.photo_url` is a computed field, while `plants.photo_storage_path` is the real column.
4. **Async Everything**: Use `async`/`await` for all DB interactions, API calls, and HTTP responses. 
5. **OpenAI Structured Outputs**: If modifying AI outputs, continue using the `response_format` pattern and test against live APIs, not just mocks.
6. **Graceful Degradation**: Continue to support fallback modes if Firebase credentials or MQTT configurations are missing, as established in the current code.

---

# AGENTS.md - Frontend Guide

This document is designed to provide context and guidance for any AI agent or developer interacting with the `frontendGossipGarden` repository. It provides an analysis of the structure, the general context of its functions, and rules for extending, refactoring, or migrating code.

## 1. Architectural Structure

The frontend is a **Flutter** application utilizing **Riverpod** for state management and following a feature-based folder structure. 

### Directory Breakdown
- **`lib/main.dart`**: The root of the application, configuring the `ProviderScope` and wrapping the UI in a global NavigationState layout (a custom routing strategy utilizing `IndexedStack` rather than `Navigator.push`).
- **`lib/core/`**: Shared infrastructure and utilities.
  - `config/`: Compile-time configurations (e.g., `AppConfig`).
  - `services/`: Low-level services such as JWT storage (`TokenStorage`) and base authentication logic against the backend (`backend_auth_service.dart`).
  - `observers/`: E.g., `SessionObserver`, which monitors globally for `UnauthorizedException` and triggers sign-outs.
- **`lib/features/`**: The core application modules organized by feature (`auth`, `plants`, etc.). Each feature contains:
  - `data/`: Models, Enums, and Datasources (HTTP Clients).
  - `presentation/`: Riverpod Providers, Screens, and custom Widgets.

## 2. General Context of Key Functions

- **Auth Lifecycle (`features/auth/presentation/providers/auth_provider.dart`)**:
  Manages the session state. Bypasses Firebase Auth in favor of direct Supabase token generation via `backend_auth_service`. Uses `flutter_secure_storage` for token persistence. Exposes `backendTokenProvider` which is reactively watched by all API datasources.
- **Plant Data Loading (`plant_api_datasource.dart` & Providers)**:
  `plantsProvider` polls for plant updates and relies on backend-side joins (common and scientific names are fetched dynamically). `plantRealtimeSensorProvider` streams sensor data dynamically.
- **Plant Identification Flow (`features/plants/presentation/screens/plant_identify_screen.dart`)**:
  Handles local camera usage, image processing (orientation, compression to 1024x1024), and coordinates the multi-step UI flow: uploading -> processing -> selecting candidates -> confirming -> creating.
- **Chat Experience**:
  State is managed by `chatMessagesProvider`. Reads directly from the `/chat` endpoints. Does not cache locally in Firestore; completely defers to the backend's memory logic.
- **Navigation (`MainScreen` and Overlays)**:
  Uses an `IndexedStack` and overlay widgets for navigation to preserve state locally without complex routing stacks. Back behavior is manually handled via `notifier.handleBack()`.

## 3. Preparation for Migrations, Refactoring, and New Features

When refactoring or adding functionalities, strictly adhere to these rules:

1. **State Management**: Use **Riverpod**. Wrap API calls in `FutureProvider` or `StateNotifierProvider`. Watch `backendTokenProvider` in all datasources to handle token refreshes automatically.
2. **Feature Isolation**: Place new features in `lib/features/<new_feature>/`. Do not bleed domain models into the global space unless strictly necessary.
3. **No Repositories Layer**: Continue the existing pattern of connecting UI Providers directly to Datasources. Avoid adding a repository abstraction layer unless logic complexity severely demands it.
4. **Auth Flow**: Do NOT integrate Firebase Auth. All auth is handled via Supabase keys and the custom backend API. If a datasource gets a 401, it must throw an `UnauthorizedException` so the `SessionObserver` can gracefully handle it.
5. **No Code Generation**: Models are hand-written. Avoid introducing `build_runner` or `freezed` unless globally agreed upon. 
6. **UI Conventions**: Keep screens lightweight and split logic into smaller widgets. Ensure user-facing text remains in Spanish. Use `NetworkImage` for plant photos via `photo_url` provided by the API.

---

# QA / Demo Environment Rules

## Sensor Hardware Mocks & Branches
Actualmente solo existe **un único sensor físico** (ESP32). Para permitir que este único sensor asigne datos telemétricos (temperatura, luz, humedad) a **todas** las plantas creadas en la base de datos de manera simultánea, se implementó un modo "Broadcast" en el backend.

- **Rama Activa para Demo/QA:** `feature/single-sensor-broadcast` (Backend)
- **Comportamiento:** El backend ignora el `plant_id` fijo (`e1db0480-8b23-4302-a752-405a45d311b5`) que envía el código C++/MicroPython del ESP32, y en su lugar consulta todas las plantas existentes en Supabase e inserta la misma lectura en Firebase para cada una de ellas.
- **Regla:** Si estás trabajando en la Demo o QA y requieres hacer cambios al backend que impliquen al sensor, asume que esta rama es la principal para el backend. No intentes modificar el código del ESP32 físico a menos que se te pida explícitamente.

---

# AGENTS.md - Presentation Guide

This document is designed to provide context and guidance for any AI agent or developer interacting with the `gossip-garden-presentation` repository.

## 1. Architectural Structure

The presentation is a **Next.js** web application (App Router) using **React** and **Tailwind CSS v4**.

### Directory Breakdown
- **`app/`**: Next.js App Router structure. Contains `layout.tsx` (Root layout, fonts), `globals.css` (Tailwind theme and custom CSS classes), and page routes.
- **`components/`**: Reusable React components. Contains `ui/` for Shadcn UI components and other custom interactive elements.
- **`lib/`**: Utility functions (e.g., `utils.ts` for Tailwind `cn` merging).
- **`public/`**: Static assets.

## 2. General Context & Design System

- **Design System ("Crayon Storybook")**: The presentation strictly shares the visual identity of the Flutter app. It uses warm paper backgrounds (`bg-background` in cream), thick brown outlines (`border-border`), and rounded, playful shapes. **Emojis are strictly forbidden** (use `lucide-react` instead). 
- **Tailwind v4 Configuration**: The theme variables and design tokens are defined inline in `app/globals.css` (using `@theme inline`), not in a `tailwind.config.ts`.
- **Custom Utility Classes**: Important utility classes exist in `globals.css` like `.crayon-card`, `.crayon-text`, `.crayon-underline`, `.breathe`, `.sway`, and `.wiggle`. These should be favored to maintain the "storybook" aesthetic.

## 3. Rules for Extending and Modifying

1. **Strictly adhere to the design skill**: Always rely on the `presentation-storybook-design` skill when modifying the UI.
2. **Next.js App Router Conventions**: Use React Server Components by default. Add `'use client'` only at the top of components that require hooks (`useState`, `useEffect`, etc.) or browser APIs.
3. **Use Semantic Tokens**: Rely on Tailwind classes referencing CSS variables for colors (e.g., `bg-card`, `text-foreground`, `bg-primary`). Do not use raw hex codes or default colors like `bg-white` or `text-black`.
4. **Motion/Animations**: Use the custom CSS animation classes in `globals.css` for micro-animations. For more complex layout transitions or scroll reveals, use `motion` (Framer Motion) with spring physics.
