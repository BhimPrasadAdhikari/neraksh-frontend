# ARCHITECTURE.md — NE India Landslide Early-Warning System

Companion to `DESIGN.md` and `IMPLEMENTATION_PLAN.md`. This file is the source of truth
for **system architecture and folder structure**, and — since `DESIGN.md` is scoped to
UI/UX only for this project — it is also the source of truth for **auth/user-profile
architecture and other product-level structural decisions** that don't have a UI/UX
home. `DESIGN.md` remains authoritative for anything it already specifies (frontend
framework, visual design, notification channels, severity thresholds, etc.); where this
file and `DESIGN.md` conflict on something `DESIGN.md` actually covers, `DESIGN.md`
wins and this file should be updated to match.

---

## 1. System Overview

Two independently-trained models answer two different questions, and the architecture
exists mainly to keep that distinction intact all the way to the API layer:

- **Susceptibility model** (`model_v3`, XGBoost) — static terrain features (elevation,
  slope, aspect, curvature, relief, roughness). Answers **where** a landslide can happen.
  Does not depend on date. Recomputing it is expensive (DEM tile reads) but rare —
  terrain doesn't change day to day.
- **Trigger model** (`model_trigger`, XGBoost) — antecedent rainfall (1/3/7/15/30-day
  windows) + NDVI + elevation. Answers **when**. Depends on `event_date`. Cheap to
  recompute per point, but the inputs come from rate-limited external APIs (NASA POWER,
  MODIS/ORNL), so it's fetched on a schedule and cached, not live per user request.

`combined_risk = susceptibility × trigger`, per point.

**The core architectural decision**: user-facing reads must never trigger live model
inference or live external API calls. All of that happens in a background pipeline that
populates a `risk_grid` table; the API layer only ever reads from Postgres for normal
traffic. One explicit, rate-limited, clearly-documented exception exists for on-demand
single-point assessment (see §4.3).

**Current-phase note on auth**: real authentication is *not* being built yet. "Login" in
this phase means selecting a hardcoded user profile (region + role); see §6 for the
full explanation. The diagram and folder structure below still show an `auth`
route/module because it will be needed eventually (Phase 8 of the implementation plan),
but for now it's a thin stand-in, not a real identity system.

```
┌─────────────┐      ┌──────────────────────────────────────────┐
│  Frontend   │◄────►│                FastAPI                    │
│ (web/mobile)│ REST │  - fast reads from risk_grid (Postgres)    │
└─────────────┘      │  - on-demand /assess (the one live route) │
                      │  - hardcoded-profile "login", incidents,  │
                      │    alerts CRUD (real RBAC deferred)        │
                      └───────────────┬────────────────────────┘
                                       │
                      ┌────────────────▼────────────────┐
                      │           Postgres + PostGIS      │
                      │  risk_grid, rainfall_ndvi_cache,  │
                      │  incidents, alerts, users          │
                      │  (users = small hardcoded seed set)│
                      └────────────────▲────────────────┘
                                       │ upserts
                      ┌────────────────┴────────────────┐
                      │   Background job scheduler        │
                      │  (Celery+Redis / APScheduler —    │
                      │   confirm in DESIGN.md)            │
                      │  - terrain feature extraction      │
                      │    (one-time / rare, per grid pt)  │
                      │  - rainfall+NDVI refresh (~6h)     │
                      │  - model scoring → risk_grid       │
                      │  - threshold diff → alerts          │
                      └────────────────┬────────────────┘
                                       │
                      ┌────────────────▼────────────────┐
                      │        External data sources      │
                      │  Copernicus DEM (S3, /vsicurl/)   │
                      │  NASA POWER (rainfall)             │
                      │  MODIS/ORNL (NDVI)                 │
                      └───────────────────────────────────┘
```

---

## 2. Backend Folder Structure

```
backend/
├── app/
│   ├── main.py                     # FastAPI app instance, lifespan hook
│   ├── config.py                   # settings (pydantic-settings, reads .env)
│   │
│   ├── api/
│   │   ├── deps.py                 # shared FastAPI dependencies (DB session,
│   │   │                           #   get_current_profile — reads a hardcoded
│   │   │                           #   profile id, not a verified session; see §6)
│   │   └── v1/
│   │       ├── router.py           # aggregates all v1 routers
│   │       ├── risk.py             # GET /risk, GET /risk/grid, POST /risk/assess
│   │       ├── incidents.py        # POST/PATCH /incidents
│   │       ├── alerts.py           # GET /alerts
│   │       ├── profiles.py         # GET /profiles (list hardcoded profiles),
│   │       │                       #   POST /select-profile ("login" stand-in — see §6)
│   │       └── health.py           # GET /health
│   │
│   ├── models/                     # SQLAlchemy ORM models (DB tables)
│   │   ├── risk_grid.py
│   │   ├── rainfall_ndvi_cache.py
│   │   ├── incident.py
│   │   ├── alert.py
│   │   └── user.py                 # seeded/hardcoded rows for now — see §6
│   │
│   ├── schemas/                    # Pydantic request/response schemas
│   │   ├── risk.py
│   │   ├── incident.py
│   │   ├── alert.py
│   │   └── profile.py              # hardcoded-profile shape (region, role)
│   │
│   ├── services/                   # business logic, framework-agnostic
│   │   ├── terrain_features.py     # DEM tile reads → slope/aspect/curvature/etc.
│   │   ├── rainfall_ndvi.py        # NASA POWER + MODIS fetch, cache-aware
│   │   ├── model_registry.py       # loads + serves both XGBoost models
│   │   ├── risk_scoring.py         # combines terrain+trigger features → combined_risk
│   │   └── alerting.py             # threshold diffing → alert creation
│   │
│   ├── ml/
│   │   ├── artifacts/              # .joblib files + *_feature_cols.json (gitignored;
│   │   │                           #   fetched from wherever DESIGN.md says at deploy)
│   │   └── loader.py               # low-level joblib load + feature-order validation
│   │
│   ├── jobs/                       # background/scheduled tasks
│   │   ├── scheduler.py            # Celery app or APScheduler config
│   │   ├── refresh_grid.py         # the periodic rainfall/NDVI/scoring job
│   │   └── dispatch_alerts.py      # picks up new alerts, sends notifications
│   │
│   ├── db/
│   │   ├── session.py              # engine/session factory
│   │   ├── seed_profiles.py        # seeds the hardcoded region/role profiles — see §6
│   │   └── migrations/             # Alembic
│   │
│   └── core/
│       ├── security.py             # placeholder only in this phase — no password
│       │                           #   hashing/JWT issuance is wired up yet; kept as
│       │                           #   the future home for real auth (Phase 8)
│       └── logging.py
│
├── tests/
│   ├── unit/                       # services/ tested in isolation, HTTP mocked
│   ├── integration/                # API routes against a seeded test DB
│   └── conftest.py
│
├── scripts/
│   └── seed_grid.py                # one-off: generate initial scoring grid points
│
├── docker-compose.yml
├── Dockerfile
├── requirements.txt / pyproject.toml
└── .env.example
```

**Why `services/` is separate from `api/`:** routes should stay thin (parse request →
call a service → serialize response). This is what makes Phase 3 of the implementation
plan (feature extraction) independently unit-testable without spinning up FastAPI, and
what lets Phase 6's background job reuse `services/model_registry.py` and
`services/rainfall_ndvi.py` without going through HTTP at all.

**Why `ml/` is separate from `services/model_registry.py`:** `ml/loader.py` is the dumb,
low-level "open this joblib file, validate its feature list" logic; `model_registry.py`
is the app-facing singleton (loaded once at startup, exposed via FastAPI `lifespan`)
that other services call. Keeping the split makes the loader testable without the app
context.

---

## 3. Frontend Folder Structure (skeleton — adjust once DESIGN.md names the framework)

```
frontend/
├── src/
│   ├── pages/ (or routes/, per framework convention)
│   │   ├── MapView/            # consumes GET /risk/grid?bbox=
│   │   ├── LocationDetail/     # consumes GET /risk
│   │   ├── ReportIncident/     # POST /incidents
│   │   ├── AdminDashboard/     # alerts list, incident verification
│   │   └── ProfileSelect/      # "login" screen — pick a hardcoded region/role
│   │                           #   profile from a list; see §6. Replaces a real
│   │                           #   Auth/ login-signup page for this phase.
│   │
│   ├── api/
│   │   └── client.ts           # typed fetch wrapper, one function per backend route
│   │
│   ├── components/             # shared UI (map layer, severity badge, etc.)
│   ├── state/                  # per DESIGN.md's chosen state management
│   └── types/                  # shared TS types mirroring backend Pydantic schemas
│
└── (build tooling per DESIGN.md)
```

Keep `api/client.ts` as the single place that knows backend URLs/shapes — no direct
`fetch()` calls scattered through components. This matters more than usual here because
the backend response shapes (severity bands, GeoJSON grid format, and eventually real
auth tokens) are still likely to shift during early phases.

---

## 4. Key Architectural Decisions

### 4.1 Fast path / slow path split (see §1)
`GET /risk` and `GET /risk/grid` **only read Postgres**. They must never import or call
`services/model_registry.py`, `services/terrain_features.py`, or
`services/rainfall_ndvi.py` directly. This is enforced by convention (code review), not
by a technical barrier — call it out explicitly in PR review checklists.

### 4.2 Caching granularity
`rainfall_ndvi_cache` is keyed to a snapped ~0.5° cell (NASA POWER's native resolution),
not to exact lat/lon — see `services/rainfall_ndvi.py`. Terrain features are **not**
cached at a coarser grain; they're computed once per exact grid point and stored
directly on `risk_grid`, since DEM resolution (~30m) is far finer than rainfall data and
collapsing it would lose real signal.

### 4.3 The one live-inference route
`POST /risk/assess` is the single exception to §4.1 — it's allowed to call external
rainfall/NDVI APIs and run model inference synchronously, because it's a deliberate,
user-initiated, rate-limited action (e.g., an admin checking an arbitrary new site), not
a dashboard load. It upserts its result into `risk_grid` on completion so subsequent
`GET /risk` calls near that point benefit immediately. Any future route that needs live
inference should be scrutinized against this pattern rather than adding a second
ad-hoc exception.

### 4.4 Model artifact versioning
`model_registry.py` loads exactly the `.joblib` + `*_feature_cols.json` pairs at the
configured `ml/artifacts/` path. The feature-column JSON is the single source of truth
for input order — no service should hardcode a feature list; all should ask the
registry to reindex a feature dict against the loaded JSON before inference. This is
what prevents the class of bug from earlier prototyping (guessing at 7 vs. 9 features,
silent misordering) from reaching production.

### 4.5 Background job idempotency
`jobs/refresh_grid.py` must be safe to kill and restart mid-run without re-charging
external API calls for cells already fetched in that run — achieved via the
`rainfall_ndvi_cache` upsert-on-fetch pattern, keyed by `(cell_lat, cell_lon,
event_date)`, checked before every external call.

### 4.6 Explainability
`services/alerting.py` must produce a human-readable `reason` string per alert (e.g.
which rainfall window moved and by how much), not just a numeric severity — this is a
direct requirement from the product's "Explainable Risk Intelligence" design goal, not
an optional nicety.

### 4.7 Auth is deferred; hardcoded profiles stand in for it (see §6 for detail)
No route in this phase should assume a verified, credentialed session. `deps.py`'s
`get_current_profile` dependency resolves whichever hardcoded profile the client says
it's using (see §6) — it is not a security boundary and must not be treated as one in
code review. Real JWT/session auth is Phase 8 of `IMPLEMENTATION_PLAN.md`, and when it
lands, `get_current_profile` is the one place that needs to change to become a real
auth dependency; routes that already depend on it should not need rewriting, only the
dependency's internals.

---

## 5. What This File Does Not Cover

Per-phase build order, acceptance criteria, and agent prompts live in
`IMPLEMENTATION_PLAN.md`. UI/UX-level product decisions (visual design, layout,
component styling, frontend framework) belong in `DESIGN.md`. `DESIGN.md` for this
project does **not** cover auth or backend/product architecture — those decisions live
here (§6 and §4.7) until a dedicated auth design doc exists. If any of §2–§4 above
assumes something `DESIGN.md` hasn't decided (e.g. severity thresholds, notification
channels, task scheduler choice), treat the assumption here as a placeholder to be
overwritten once `DESIGN.md` is updated, not as a final decision.

---

## 6. Authentication & User Profiles — Current Phase (Hardcoded, No Real Login)

**Decision:** for this phase of the project, real authentication (signup, password/OTP
login, session/JWT issuance against a live credential store) is explicitly **out of
scope**. It's replaced by a small, hardcoded set of user profiles that a person picks
from instead of logging in. This is a deliberate simplification to unblock building the
rest of the system (map, risk API, incidents, alerts, dashboards) without auth work on
the critical path — not an oversight, and not what Phase 8 of `IMPLEMENTATION_PLAN.md`
ultimately builds.

### 6.1 What "login" means right now
- The frontend shows a **profile picker** (`ProfileSelect/`, §3) instead of a
  login/signup form: a list of the hardcoded profiles below, grouped by region and role.
- Picking a profile calls `POST /profiles/select` (or equivalent), which returns a
  simple identifier the frontend then sends back on subsequent requests (e.g. as a
  header or a non-secret cookie) — there is no password check, no token signing, and no
  expiry/refresh logic. This identifier only tells the backend *which hardcoded profile
  to behave as*, not that the caller has proven who they are.
- `api/deps.py`'s `get_current_profile` dependency reads that identifier and looks up
  the matching row from the hardcoded seed set (§6.2) — it does not verify a signature,
  a session store, or a credential.

### 6.2 The hardcoded profile set
Profiles are seeded via `db/seed_profiles.py` (or an equivalent fixture/migration) and
combine two axes:
- **Region** — one profile (or small set of profiles) per NE India state/area covered by
  the project: Arunachal Pradesh, Assam, Manipur, Meghalaya, Mizoram, Nagaland, Sikkim,
  Tripura. A region-scoped profile should see/interact with risk data, incidents, and
  alerts relevant to that region.
- **Role** — `admin`, `officer` (government/field), and `citizen`/`community`, mirroring
  the roles implied by the architecture diagram's Government/Citizen split. Role
  determines which routes and dashboard views a profile can reach (e.g. an officer
  profile can verify incidents; a citizen profile can only submit them).

The exact list of profiles (how many per region, whether every region gets all three
roles) is an open decision — confirm with the product owner or pin it down in
`db/seed_profiles.py` directly rather than guessing at a full matrix up front.

### 6.3 How this interacts with the rest of the architecture
- **Routes still get a "current profile"** via `get_current_profile`, so route code
  written now (incident submission, alert viewing, admin actions) doesn't need to be
  rewritten when real auth arrives — only the internals of that one dependency change.
- **RBAC enforcement is real, credential verification is not.** A route can still say
  "only `admin`/`officer` profiles can `PATCH /incidents/{id}`" and enforce that today —
  what's missing is proof that the caller actually *is* the profile it claims to be.
  Treat every hardcoded-profile session as fully trusted/unauthenticated-adjacent; this
  is not safe to expose outside a controlled demo/pilot environment.
- **`users` table** (Phase 1 schema) is populated by the seed script with the hardcoded
  profiles rather than by a signup flow. Its shape should still match what real auth
  will eventually need (id, role, region, display name) so Phase 8 can add
  password/identity fields without a structural rewrite.
- **Phase 8 of `IMPLEMENTATION_PLAN.md`** ("Auth & RBAC") is where this gets replaced:
  when it's picked up, `DESIGN.md` needs to gain the missing auth-scheme decision (JWT
  vs. session, provider), and this section should be updated to describe the real login
  flow and marked superseded rather than deleted, so the hardcoded-profile approach
  stays documented as project history.

### 6.4 Why this lives here and not in DESIGN.md
`DESIGN.md` for this project is scoped to UI/UX (visual design, layout, component
styling) and does not contain auth or backend/product-architecture decisions. Rather
than leaving the hardcoded-profile approach undocumented, it's recorded here since this
file is the source of truth for system architecture. If a dedicated auth/product design
doc is created later, this section should move there and be linked from here instead of
duplicated.
