# NE India Landslide Early-Warning System — Implementation Plan

This plan sequences the build into phases for an AI coding agent (e.g. Claude Code).
Every phase's prompt tells the agent to **read `DESIGN.md` first** and defer to it for
any architectural/design decision not explicitly pinned down here. This file pins down
*what* to build and *in what order*; `DESIGN.md` should remain the source of truth for
*how* (stack choices, naming conventions, folder layout, auth scheme, etc.).

Treat each phase as a separate agent session/task. Do not start Phase N+1 until Phase N's
acceptance criteria are met — later phases assume earlier ones exist and work.

---

## Phase 0 — Repo & Environment Scaffolding

**Goal:** A running skeleton with no business logic yet — proves the stack boots.

**Prompt for the agent:**
> Read `DESIGN.md` fully before doing anything. Set up the project skeleton it describes:
> FastAPI backend, Postgres (with PostGIS enabled), and whatever frontend framework
> `DESIGN.md` specifies. Create the folder structure `DESIGN.md` lays out (or propose one
> and confirm with me if it doesn't specify one). Add a working `docker-compose.yml` (or
> equivalent, per `DESIGN.md`) that brings up Postgres+PostGIS and the FastAPI app with a
> single `/health` endpoint. Add `.env.example` for all config the app will need (DB URL,
> external API keys/endpoints, model file paths). Do not implement any model, DB table, or
> business route yet — this phase is scaffolding only. Confirm `/health` returns 200 before
> finishing.

**Acceptance criteria:**
- `docker-compose up` (or equivalent) brings up DB + API cleanly
- `/health` returns 200
- `.env.example` covers every secret/config referenced anywhere later in this plan

---

## Phase 1 — Database Schema (Postgres + PostGIS)

**Goal:** Tables exist, migrated, matching what later phases need to read/write.

**Prompt for the agent:**
> Read `DESIGN.md` for naming conventions, migration tool choice (Alembic unless
> `DESIGN.md` says otherwise), and any schema decisions it already made. Implement these
> tables, adjusting types/names to match `DESIGN.md`'s conventions:
> - `risk_grid`: id, location (PostGIS `geography(Point)`), latitude, longitude,
>   elevation_m, slope_deg, aspect_sin, aspect_cos, curvature, relief_amplitude,
>   roughness, plan_curvature, profile_curvature, susceptibility, trigger_score,
>   combined_risk, event_date, computed_at.
> - `rainfall_ndvi_cache`: cell_lat, cell_lon, event_date, rain_1d..rain_30d, ndvi,
>   ndvi_date, fetched_at. Unique on (cell_lat, cell_lon, event_date).
> - `incidents`: id, user_id, latitude, longitude, description, media_refs (array/json),
>   status (pending/verified/rejected), created_at, verified_at.
> - `alerts`: id, risk_grid_id (nullable), severity (low/moderate/high/critical), title,
>   reason (explainability text), affected_area (geography, nullable), created_at,
>   dispatched_at.
> - `users`, `roles`: matching whatever auth/RBAC scheme `DESIGN.md` specifies.
> Write this as versioned migrations, not a single init script, so schema changes later
> in the project are trackable. Add indexes for: spatial lookups on `risk_grid.location`
> (GiST index), and the unique constraint on `rainfall_ndvi_cache`. Do not write any
> application code that uses these tables yet.

**Acceptance criteria:**
- Migrations run cleanly from empty DB
- PostGIS spatial index exists on `risk_grid.location`
- Schema reviewed against DESIGN.md's naming conventions before merging

---

## Phase 2 — Model Artifact Loading Layer

**Goal:** The two trained models (`model_v3` susceptibility, `model_trigger`) load once
at app startup and are available to request handlers without re-loading per request.

**Prompt for the agent:**
> Read `DESIGN.md` for where trained model artifacts should live (local path vs. object
> storage) and how app-wide singletons should be managed in this FastAPI app. The two
> model files are `model_v3_susceptibility.joblib` and `model_trigger.joblib`, each with
> a companion `*_feature_cols.json` pinning exact input column order — treat the JSON as
> the single source of truth for feature order, never hardcode column order elsewhere.
> Implement a `ModelRegistry` (or whatever pattern `DESIGN.md` prefers) that:
> - Loads both models + their feature-column lists once in the FastAPI `lifespan` startup
>   hook, not per-request.
> - Exposes a typed method to predict susceptibility given a dict of the 9 (or 7, pruned)
>   terrain features, and trigger probability given a dict of the 7 trigger features —
>   internally reindexing the dict to the JSON's column order before calling
>   `predict_proba`, so a caller passing keys in any order can't silently corrupt results.
> - Fails loudly at startup (not at first request) if either artifact or its JSON is
>   missing or the columns don't match what the loaded model expects.
> Do not implement any feature-extraction (DEM, rainfall, NDVI) or any API route in this
> phase — only the load-and-predict layer, testable in isolation with a hand-built
> feature dict.

**Acceptance criteria:**
- Unit test: loading with a deliberately-reordered feature dict still produces the same
  prediction as the correctly-ordered dict
- Unit test: missing/mismatched artifact fails at startup, not silently at inference time

---

## Phase 3 — Feature Extraction Services

**Goal:** Port the notebook's DEM/rainfall/NDVI extraction logic into backend services,
with the external-API calls going through the `rainfall_ndvi_cache` table from Phase 1.

**Prompt for the agent:**
> Read `DESIGN.md` for service-layer conventions (where business logic lives relative to
> routes, whether it uses a repository pattern, async vs sync DB access, etc.). Port the
> following into backend services, matching those conventions:
> - `TerrainFeatureService`: given lat/lon, reads the Copernicus DEM COG tile (same
>   `/vsicurl/` tile-URL pattern as the prototype notebook) and computes elevation, slope,
>   aspect (as sin/cos), curvature, relief amplitude, roughness, plan/profile curvature.
> - `RainfallNdviService`: given lat/lon/event_date, first checks `rainfall_ndvi_cache`
>   (snapped to the same ~0.5° cell size used in the prototype, since that's NASA POWER's
>   native resolution) before calling NASA POWER (antecedent rainfall, windows 1/3/7/15/30
>   days) and the MODIS ORNL NDVI endpoint. On a cache miss, fetch, upsert into the cache
>   table, then return. On external API failure, retry with backoff (reuse the prototype's
>   retry pattern) and return a clearly-flagged null/NaN result rather than raising, so a
>   single bad point doesn't crash a batch job.
> Ask me for the exact prototype extraction code if you need the geomorphometric formulas
> (relief amplitude, roughness, plan/profile curvature) — don't reimplement them from
> scratch or guess at the Zevenbergen & Thorne finite-difference coefficients.
> Write these as pure services with no FastAPI route wiring yet — they should be directly
> unit-testable against a few known lat/lon points.

**Acceptance criteria:**
- Both services testable in isolation (mock the external HTTP calls in tests)
- Cache-hit path never makes an external HTTP call — verify this in a test
- A deliberately invalid coordinate returns a flagged-null result, not an exception

---

## Phase 4 — Synchronous Risk API (fast path)

**Goal:** `GET /api/risk` and `GET /api/risk/grid` — reads from `risk_grid`, no live
model inference, matches the diagram's "Current Location State Engine."

**Prompt for the agent:**
> Read `DESIGN.md` for API route/response conventions (versioning, error format,
> pagination style). Implement:
> - `GET /api/risk?lat=&lon=` — nearest-neighbor PostGIS query against `risk_grid`
>   (`ST_Distance` / `<->` operator), returns susceptibility, trigger_score,
>   combined_risk, computed_at, and a severity band (low/moderate/high/critical —
>   confirm thresholds with me or check if `DESIGN.md` already defines them). Return a
>   clear "no cached data near this point yet" response rather than a 500 if the grid
>   doesn't cover that area.
> - `GET /api/risk/grid?bbox=` — returns all `risk_grid` rows within a bounding box as
>   GeoJSON FeatureCollection, for map rendering on the frontend.
> Both routes must not call the model registry or feature-extraction services directly —
> they only read `risk_grid`, which Phase 6's background job populates. Add integration
> tests against a seeded test DB with a handful of known `risk_grid` rows.

**Acceptance criteria:**
- Both routes respond in well under 200ms against a seeded local DB (no external calls,
  no model inference — this is the whole point of the split)
- Empty-grid-region case returns a clean, documented response, not an error

---

## Phase 5 — On-Demand Assessment API (slow path)

**Goal:** `POST /api/risk/assess` — the one endpoint allowed to run live inference,
for a single arbitrary point not yet in the grid (matches diagram's on-demand
"Landslide Risk Engine" trigger).

**Prompt for the agent:**
> Read `DESIGN.md` for background-task/async conventions. Implement
> `POST /api/risk/assess { lat, lon, event_date? }` that:
> - Calls `TerrainFeatureService` and `RainfallNdviService` from Phase 3 live
> - Feeds results through `ModelRegistry` from Phase 2 to get susceptibility, trigger,
>   and combined_risk (susceptibility × trigger)
> - Returns the result immediately in the response (this endpoint is explicitly allowed
>   to be slower than Phase 4's routes, since it's a deliberate one-off action), and also
>   upserts the result into `risk_grid` so future nearby `GET /api/risk` calls benefit
> - Rate-limit or otherwise guard this route per `DESIGN.md`'s policy, since it's the one
>   path that can generate real external-API load per request — ask me if `DESIGN.md`
>   doesn't specify a policy.
> Document in the route's docstring/OpenAPI description that this is the *only*
> synchronous route permitted to call external APIs, so future contributors don't add
> more without thinking about latency.

**Acceptance criteria:**
- Response includes both the live result and confirmation it was persisted to `risk_grid`
- A second `GET /api/risk` call for a nearby point after this now returns the new data

---

## Phase 6 — Background Grid Refresh Job

**Goal:** Scheduled job that keeps `risk_grid` current, matching the diagram's ingestion
→ processing engines running continuously behind the scenes.

**Prompt for the agent:**
> Read `DESIGN.md` for the chosen task scheduler (Celery+Redis, APScheduler, or other —
> don't assume, check first) and its configured schedule/retry conventions. Implement a
> job that:
> - On first run (or when new grid points are added), computes static terrain features
>   for every point in the scoring grid via `TerrainFeatureService` and stores them —
>   these never need recomputation unless the DEM source changes, so separate this from
>   the rainfall/NDVI refresh, which does need to rerun regularly.
> - On its recurring schedule (confirm interval with me — antecedent rainfall windows go
>   up to 30 days, so refreshing more often than every few hours has limited value; every
>   6 hours is a reasonable default, but check `DESIGN.md` or ask), re-fetches
>   rainfall/NDVI for the grid's unique 0.5° cells via `RainfallNdviService`, re-scores
>   every grid point through `ModelRegistry`, and upserts into `risk_grid`.
> - After each refresh, diffs new `combined_risk` values against the previous run's
>   values per point; where a point crosses a severity threshold upward, writes a row to
>   `alerts` with an explainability `reason` string (e.g. "rain_7d rose from Xmm to Ymm,
>   pushing trigger score from A to B" — keep it human-readable per the diagram's
>   "Explainable Risk Intelligence" box).
> - Logs enough (points processed, cache hit rate, failures) to debug a bad run without
>   re-running it.
> This job must be resumable/idempotent — if it dies partway through a grid refresh, a
> retry should not double-charge external API calls for cells already fetched this run.

**Acceptance criteria:**
- Job runs end-to-end against a small test grid (not the full ~34k-point production grid)
  in CI or local test
- Killing and restarting the job mid-run doesn't duplicate API calls for already-cached
  cells in that run
- At least one seeded "crosses threshold" scenario produces an `alerts` row with a
  populated `reason`

---

## Phase 7 — Incident Reporting & Feedback Loop

**Goal:** Citizen-submitted reports (diagram's "Report Incident" + "Feedback &
Verification Loop"), feeding back into the system rather than sitting inert.

**Prompt for the agent:**
> Read `DESIGN.md` for file/media upload conventions (storage backend for photos/videos)
> and the auth/RBAC scheme for who can verify a report. Implement:
> - `POST /api/incidents` — citizen submits lat/lon, description, media; stored with
>   status `pending`.
> - `PATCH /api/incidents/{id}` — authorized field/admin user marks
>   verified/rejected, per `DESIGN.md`'s RBAC rules.
> - On verification, optionally trigger a Phase 5-style on-demand reassessment of that
>   location (confirm with me whether every verified incident should trigger this, or
>   only ones that materially disagree with the current cached risk — the diagram's
>   "Reassess Risk" step in the Feedback & Verification Loop implies this should happen,
>   but the exact trigger condition needs a decision).
> Do not build photo/video hazard analysis (the diagram's "Field Image Analysis Engine")
> in this phase — that's a separate model/service, out of scope until we decide whether
> to build it.

**Acceptance criteria:**
- Full lifecycle (submit → pending → verify → status update) testable end-to-end
- Verification triggers whatever reassessment behavior was agreed, and it's observable
  in `risk_grid`/`alerts`

---

## Phase 8 — Auth & RBAC

**Goal:** Diagram's "Login/Sign-up → Authorization (RBAC) → Government/Citizen" split,
enforced on every route added in prior phases.

**Prompt for the agent:**
> Read `DESIGN.md` for the exact auth scheme (JWT, session, provider) already decided.
> Implement login/signup and role-based access control matching it, then retrofit every
> route from Phases 4, 5, and 7 with the correct role requirements (e.g. citizens can
> call `GET /api/risk` and `POST /api/incidents` but not `PATCH /api/incidents/{id}`;
> government/admin roles can access dashboards and verification routes). Do not change
> any route's business logic — only add auth dependencies. Add tests asserting each
> route rejects insufficiently-privileged callers.

**Acceptance criteria:**
- Every route from Phases 4/5/7 has an explicit, tested role requirement — no route left
  implicitly open by omission

---

## Phase 9 — Alerts & Notification Dispatch

**Goal:** Turn `alerts` rows (written by Phase 6) into actual outbound notifications.

**Prompt for the agent:**
> Read `DESIGN.md` for which notification channels are in scope for this version (SMS,
> push, dashboard-only, etc. — the diagram shows SMS/App Push/Dashboard/Community, but
> confirm which are actually being built now vs. later). Implement a dispatcher that
> picks up new `alerts` rows and sends them via the confirmed channel(s), marking
> `dispatched_at` once sent. Keep the dispatcher decoupled from Phase 6's scoring job —
> it should be triggerable independently (e.g. its own small scheduled poll, or a
> Postgres LISTEN/NOTIFY trigger if `DESIGN.md` prefers event-driven over polling).

**Acceptance criteria:**
- A manually-inserted `alerts` row gets dispatched and `dispatched_at` populated, without
  needing to run the full Phase 6 job

---

## Phase 10 — Frontend Integration

**Goal:** Wire the frontend (framework per `DESIGN.md`) to the endpoints above.

**Prompt for the agent:**
> Read `DESIGN.md` for the frontend stack, state-management approach, and map/visualization
> library already chosen. Build, in this order: (1) the map view consuming
> `GET /api/risk/grid?bbox=` to render the heatmap, (2) the location-detail view
> consuming `GET /api/risk`, (3) the incident-report form posting to
> `POST /api/incidents`, (4) the admin dashboard's alert list. Do not build the "Ask AI"
> assistant or the Field Image Analysis upload flow yet — those depend on services not
> built in this plan. Match `DESIGN.md`'s component/folder conventions.

**Acceptance criteria:**
- All four views work against the Phase 0-9 backend running locally
- Map correctly reflects severity bands from Phase 4's thresholds

---

## Explicitly Out of Scope for This Plan

These appear in the architecture diagram but are **not** covered by phases above —
call this out to the agent so it doesn't try to improvise them:
- Satellite Change Engine (surface/slope change detection)
- Field Image Analysis Engine (photo/video hazard detection)
- "Ask AI" assistant
- Multilingual alert content
- Response Prioritization / Response & Action Engine (resource dispatch logic)

If any of these get greenlit later, they should get their own phase(s) appended to this
plan, each still starting with "read `DESIGN.md` first."

---

## How to Use This File With the Agent

For each phase, give the agent:
1. This phase's section from this file, verbatim
2. A reminder: *"Read DESIGN.md before writing any code. If something in this phase
   prompt conflicts with DESIGN.md, stop and ask rather than picking one."*
3. Access to the previous phase's code/tests as context

Do not paste the whole plan into one session — phases are sized to be separate agent
tasks so context stays focused and reviewable.
