# NE India Landslide Early-Warning System — Implementation Plan

This plan sequences the build into phases for an AI coding agent. Every phase's prompt tells the agent to **read `DESIGN.md` and `ARCHITECTURE.md` first** and defer to them for any architectural/design decision not explicitly pinned down here.

Treat each phase as a separate agent session/task. Do not start Phase N+1 until Phase N's acceptance criteria are met — later phases assume earlier ones exist and work.

**Docker Rule**: Every phase must execute, test, and validate inside Docker containers (`docker compose up`). Do not rely on host-level Python or Node.js runtimes.

---

## Phase 0 — Dockerized Repo & Environment Scaffolding

**Goal:** A fully dockerized running skeleton with live-reloading backend and health checks — proves the container stack boots.

**Prompt for the agent:**
> Read `DESIGN.md` and `ARCHITECTURE.md` fully before doing anything. Set up the containerized project skeleton:
> FastAPI backend, Postgres with PostGIS enabled, and a lightweight Node/Frontend scaffolding matching `DESIGN.md`. 
> Create the directory layout matching `ARCHITECTURE.md`.
> Create a comprehensive `docker-compose.yml` that brings up:
> 1. `db`: Postgres + PostGIS with persistent volume mounts and health check (`pg_isready`).
> 2. `api`: FastAPI application with hot-reloading (`uvicorn --reload`), mounting `./backend` code into the container.
> 3. `redis`: Redis instance for background tasks/caching (if specified in `ARCHITECTURE.md`/`DESIGN.md`).
> Add `.env.example` for all config the app will need (DB URL, external API keys, model file paths).
> Ensure a single `/health` endpoint exists on FastAPI that verifies DB connectivity.
> Confirm `docker compose up` brings up all services cleanly and `/health` returns HTTP 200 before finishing.

**Acceptance criteria:**
- `docker compose up` brings up DB, API, and Redis cleanly without host dependencies.
- `curl http://localhost:8000/health` inside or outside the container returns 200 with DB health verified.
- `.env.example` covers all environment variables required by all services.

---

## Phase 1 — Database Schema & Dockerized Migrations

**Goal:** PostGIS spatial tables exist, migrated via Alembic inside Docker.

**Prompt for the agent:**
> Read `DESIGN.md` and `ARCHITECTURE.md` for schema specifications and naming conventions.
> Configure Alembic migrations inside the `api` container. Implement the following tables:
> - `risk_grid`: id, location (PostGIS `geography(Point)`), latitude, longitude, elevation_m, slope_deg, aspect_sin, aspect_cos, curvature, relief_amplitude, roughness, plan_curvature, profile_curvature, susceptibility, trigger_score, combined_risk, event_date, computed_at.
> - `rainfall_ndvi_cache`: cell_lat, cell_lon, event_date, rain_1d..rain_30d, ndvi, ndvi_date, fetched_at. Unique on (cell_lat, cell_lon, event_date).
> - `incidents`: id, user_id, latitude, longitude, description, media_refs (array/json), status (pending/verified/rejected), created_at, verified_at.
> - `alerts`: id, risk_grid_id (nullable), severity (low/moderate/high/critical), title, reason (explainability text), affected_area (geography, nullable), created_at, dispatched_at.
> - `users`, `roles`: matching hardcoded seed user structures per `ARCHITECTURE.md` §6.
> Write versioned migrations. Add spatial GiST indexes on `risk_grid.location` and unique constraints on `rainfall_ndvi_cache`.
> Add an entrypoint script or container command to auto-run migrations (`alembic upgrade head`) and seed hardcoded profiles (`db/seed_profiles.py`) upon DB container readiness.

**Acceptance criteria:**
- `docker compose run --rm api alembic upgrade head` executes without errors on a fresh PostGIS container.
- Spatial indexes exist on `risk_grid.location`.
- Hardcoded profiles are seeded in the database upon startup.

---

## Phase 2 — Docker-Compatible Model Artifact Loading Layer

**Goal:** The two trained models (`model_v3` susceptibility, `model_trigger`) load safely within the container environment during startup.

**Prompt for the agent:**
> Read `ARCHITECTURE.md` §2 and §4.4. Ensure model artifacts (`model_v3_susceptibility.joblib`, `model_trigger.joblib`) and their companion `*_feature_cols.json` files are properly mounted via volume or path configuration in `docker-compose.yml` into `app/ml/artifacts/`.
> Implement `ml/loader.py` and `services/model_registry.py` that:
> - Loads both models + feature column order dynamically in the FastAPI `lifespan` hook during container boot.
> - Validates column order matching the feature JSON and exposes typed prediction methods.
> - Fails startup immediately if model artifacts are corrupted or missing from the mounted container directory.
> Add Dockerized unit tests (`docker compose run --rm api pytest tests/unit/test_model_registry.py`) testing feature reordering and failure states.

**Acceptance criteria:**
- API container boots successfully with mounted model artifacts.
- Unit tests run and pass inside a Docker container via `pytest`.

---

## Phase 3 — Feature Extraction Services & Caching

**Goal:** DEM/rainfall/NDVI extraction services operating with containerized caching (`rainfall_ndvi_cache`).

**Prompt for the agent:**
> Read `ARCHITECTURE.md` §2 for service layout. Implement:
> - `TerrainFeatureService`: Reads DEM tiles via `/vsicurl/` over network and computes 9 geomorphometric features.
> - `RainfallNdviService`: Fetches NASA POWER / MODIS rainfall & NDVI data, using `rainfall_ndvi_cache` table to prevent duplicate fetches.
> Ensure all external requests handle network retries, timeouts, and fallback graceful failures gracefully.
> Write isolated Dockerized unit/integration tests (`docker compose run --rm api pytest tests/unit/test_services.py`) using mocked HTTP calls for external APIs.

**Acceptance criteria:**
- Tests execute inside the `api` Docker container and pass.
- Cache hits avoid external HTTP calls.
- Failed external API requests return flagged-null results rather than crashing the process.

---

## Phase 4 — Synchronous Fast-Path Risk API

**Goal:** `GET /api/risk` and `GET /api/risk/grid` reading directly from PostGIS inside Docker without running inference.

**Prompt for the agent:**
> Implement Fast-Path routes per `ARCHITECTURE.md` §4.1:
> - `GET /api/risk?lat=&lon=`: Fast PostGIS nearest-neighbor spatial query.
> - `GET /api/risk/grid?bbox=`: Spatial bounding-box query returning GeoJSON FeatureCollection.
> Ensure responses conform strictly to `DESIGN.md` risk severity definitions.
> Write Dockerized integration tests in `tests/integration/test_risk_routes.py` verifying response times (<200ms) and spatial output formatting.

**Acceptance criteria:**
- Endpoint tests pass inside Docker against a seeded test database.
- Zero calls to external APIs or model registry from these routes.

---

## Phase 5 — On-Demand Live Assessment API (Slow Path)

**Goal:** `POST /api/risk/assess` endpoint performing live inference and persisting results into `risk_grid`.

**Prompt for the agent:**
> Implement the single synchronous live inference endpoint `POST /api/risk/assess` per `ARCHITECTURE.md` §4.3:
> - Calls `TerrainFeatureService` and `RainfallNdviService` live.
> - Evaluates risk via `ModelRegistry`.
> - Upserts results into `risk_grid` and returns the immediate risk assessment.
> Limit rate per IP/User to prevent API abuse.
> Write Docker integration tests ensuring live inference completes and persistency in PostGIS is verified.

**Acceptance criteria:**
- `POST /api/risk/assess` runs live inference inside the container and saves to PostGIS.
- Subsequent `GET /api/risk` queries pick up the updated score immediately.

---

## Phase 6 — Containerized Background Scheduler & Grid Refresh Job

**Goal:** Periodic scoring pipeline running in a dedicated Celery/APScheduler container.

**Prompt for the agent:**
> Read `ARCHITECTURE.md` §1 & §2. Add a `worker` service in `docker-compose.yml` running the background scheduler:
> - One-off computation: Calculates static DEM features for grid points via `TerrainFeatureService`.
> - Scheduled refresh (~6h): Re-fetches rainfall/NDVI, re-scores grid points via `ModelRegistry`, and upserts into `risk_grid`.
> - Alerting: Diffs new `combined_risk` values against prior state; creates `alerts` rows with human-readable explainability strings (`reason`) on upward threshold crosses.
> Ensure job execution is idempotent and container restarts resume gracefully without repeating cached external API requests.

**Acceptance criteria:**
- `worker` container boots alongside `api` in `docker-compose.yml`.
- Scheduled execution processes grid points and populates `alerts` with explainable reason strings upon risk escalations.

---

## Phase 7 — Incident Reporting & Feedback Loop

**Goal:** Incident submission and field verification routes.

**Prompt for the agent:**
> Implement incident reporting routes per `ARCHITECTURE.md` §2:
> - `POST /api/incidents`: Submit incident with description, location, and media references.
> - `PATCH /api/incidents/{id}`: Update incident verification status (restricted to admin/officer profiles per `ARCHITECTURE.md` §6).
> - Trigger an automatic reassessment (`POST /api/risk/assess` logic) for verified incident locations.
> Store media references appropriately (e.g. mock/local volume upload endpoint inside Docker).

**Acceptance criteria:**
- Complete incident lifecycle (submit → verify → trigger reassessment) passes in Docker integration tests.

---

## Phase 8 — Profile-Based RBAC & Stand-in Auth

**Goal:** Enforce hardcoded user profile selection and role permissions across all routes.

**Prompt for the agent:**
> Read `ARCHITECTURE.md` §6. Implement `POST /api/profiles/select` and `deps.py:get_current_profile`:
> - Resolve profile selection (Region + Role: Admin, Officer, Citizen) based on incoming headers or profile cookies.
> - Enforce RBAC middleware/dependencies across all routes (e.g., Citizen can create incidents but cannot verify them or trigger admin workflows).
> Add unit/integration tests confirming unauthorized requests return HTTP 403.

**Acceptance criteria:**
- Route permissions are strictly enforced based on selected hardcoded profiles inside Docker test suites.

---

## Phase 9 — Alert Dispatcher Service

**Goal:** Outbound alert dispatch system.

**Prompt for the agent:**
> Build a dedicated background dispatcher task (in the `worker` container) that polls/subscribes to un-dispatched `alerts` rows and processes them, setting `dispatched_at` timestamp.
> Ensure alert formatting aligns with `DESIGN.md` §18 & §19 notification rules.

**Acceptance criteria:**
- `alerts` rows receive `dispatched_at` timestamps upon worker execution without blocking API request threads.

---

## Phase 10 — Containerized Frontend & UI Integration

**Goal:** Build and serve the frontend web app fully containerized.

**Prompt for the agent:**
> Read `DESIGN.md` for UI specifications and design system tokens. Add a `frontend` service to `docker-compose.yml` (e.g., Vite/React or Next.js dev server with hot reloading):
> - Implement Map View (`GET /api/risk/grid`), Location Detail (`GET /api/risk`), Incident Report Form (`POST /api/incidents`), and Profile Selector (`/profiles`).
> - Apply design tokens, Lucide icons, and semantic risk scales (`#159447`, `#D9A441`, `#E57A17`, `#C92A2A`) per `DESIGN.md` §4.3.
> Ensure proxying/CORS works seamlessly between `frontend` and `api` containers.

**Acceptance criteria:**
- Running `docker compose up` brings up the entire system (DB, API, Worker, Redis, Frontend).
- Navigating to `http://localhost:3000` (or designated port) renders the risk map and functional UI components interacting cleanly with the backend API container.