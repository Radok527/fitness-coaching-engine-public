# Fitness Coaching Engine

![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.134-009688.svg)
![Uvicorn](https://img.shields.io/badge/Uvicorn-ASGI-green.svg)
![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL_16-336791.svg)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED.svg)
![JWT](https://img.shields.io/badge/Auth-JWT-black.svg)
![Deterministic-Core](https://img.shields.io/badge/Analytics-Deterministic-critical.svg)
![LLM](https://img.shields.io/badge/LLM-Ollama-orange.svg)
![React](https://img.shields.io/badge/React-19-61DAFB.svg)
![PWA](https://img.shields.io/badge/PWA-Workbox-5A0FC8.svg)

A deterministic strength training analytics engine combining semantic
exercise normalization with a structured relational performance database.

---

# Overview

Fitness Coaching Engine is a backend-driven strength analytics system that combines:

- Hybrid semantic exercise normalization (embedding + rule-based)
- Controlled semantic expansion via review gating
- Deterministic workout persistence
- Structured relational training model
- Rolling structural analytics
- JWT-based authentication
- Multi-worker concurrency support
- Clean separation between AI and computation layers

AI is used for normalization and future interpretation — never for core metric computation.

---

# Architecture

The system follows a layered backend architecture:

```
backend/
  app/
    api/              → FastAPI route layer
    services/         → Business logic layer
    infrastructure/   → Repository & persistence layer
    analytics/        → Deterministic metric engine
    core/             → Ontology, mappings, normalization logic
    importers/        → CSV ingestion logic
    auth/             → JWT security & dependencies
  cli/                → CLI tools (setup, review, import)
frontend/             → React + Vite frontend
```

---

# Logical Layer Separation

## 1️⃣ Exercise Knowledge Layer

Responsible for semantic identity and normalization.

- Canonical exercise identity (`canonical_name`)
- User-facing display name (`display_name`)
- Embedding stored per exercise row
- Hybrid escalation-based classification
- Deterministic keyword overrides
- Token-overlap validation guard
- Translation fallback via LLM
- Confidence scoring
- Review-gated semantic expansion

No alias merging.  
No identity collapsing.  
Each language variant exists as its own row.

Only trusted exercises participate in similarity matching.

---

## 2️⃣ Training Analytics Layer

Fully deterministic metric computation.

- Rolling hard-set volume aggregation
- Push / Pull / Lower classification
- Movement pattern tracking
- Muscle group balance analysis
- Estimated 1RM calculation (Epley)
- Structured relational progression model
- Linear trend modeling
- Plateau detection
- Conservative adaptive progression suggestions

All analytics are reproducible and independent of AI.

---

## 3️⃣ Authentication Layer (JWT)

The system uses stateless JWT authentication with full account lifecycle support.

- Register endpoint
- Login endpoint (OAuth2PasswordRequestForm)
- Bearer token authentication
- Refresh token support (SHA-256 hashed, revocable, 30-day expiry)
- Token cleanup on startup
- Email verification flow (required before login)
- Password reset flow (secure token-based)
- Protected training & analytics routes
- Subject stored as string (JWT compliant)
- Configurable token expiration
- Password hashing via Argon2
- Rate limiting via Slowapi

Authentication is fully separated from business logic.

---

## 4️⃣ API Layer (FastAPI)

- REST interface for frontend
- OpenAPI / Swagger auto-generated
- Strict separation from analytics logic
- No business logic inside routes
- Background CSV processing
- Multi-worker support
- Security headers middleware (X-Content-Type-Options, X-Frame-Options, Referrer-Policy, Permissions-Policy)

Active route modules:

| Route | Purpose |
|---|---|
| `auth` | Register, login, refresh token, logout, password reset |
| `email_verification` | Email verification flow |
| `training` | Session logging and management |
| `analytics` | Deterministic metrics endpoints |
| `recommendation` | Adaptive load progression suggestions |
| `goals` | Goal definition, tracking, projection |
| `profile` | User profile and bodyweight/diet management |
| `review` | Exercise review and confidence gating |
| `export` | CSV data export |
| `tracked_exercises` | Per-exercise tracking metadata |

---

## 5️⃣ AI Normalization Layer

AI assists only in:

- Exercise name normalization
- Multilingual translation (async translation queue worker)
- Embedding generation
- Goal coaching recommendations
- Adaptive load progression suggestions

AI never computes training metrics.

---

## 6️⃣ PWA & Offline Support

The frontend is a Progressive Web App with full offline capability:

- Workbox Service Worker — asset and API response caching
- IndexedDB (idb) — local data persistence
- Background sync (`usePeriodicSync`) — deferred sync on reconnect
- Online/offline detection (`useOnlineStatus`) with `OfflineBanner` notification
- TanStack Query persistence — query cache survives page reload

---

## 7️⃣ Internationalization (i18n)

The UI is fully localized in German and English:

- i18next + react-i18next
- Language toggle with browser preference fallback
- LLM-based translation fallback for exercise names via Ollama
- Async translation queue worker for background processing

---

## 8️⃣ Mobile (Capacitor Android)

The app can be deployed as a native Android app:

- Capacitor Android 7.0 integration
- Mobile-first UI: bottom navigation, swipe gestures, landscape overlay guard
- Responsive layout with breakpoint-aware sidebar/drawer navigation

---

# Hybrid Classification System

Exercises are classified using an escalation-based pipeline:

```
1. Absolute Keyword Override (native)
2. Native Embedding Similarity
3. English Guard + Token Overlap Validation
4. Translation (LLM) Fallback
5. Override on Translated Text
6. Translated Embedding Similarity
7. Confidence-based Review Decision
```

## Deterministic Safeguards

- Token overlap validation prevents context-noise drift
- Similarity score is a signal — not a sole decision
- Confidence threshold controls auto-acceptance
- Review system gates semantic expansion
- Embeddings stored per exercise row
- No uncontrolled reference space growth

---

# Training Data Model

### training_sessions
- user_id
- date
- start_time
- duration_minutes
- program_name
- notes

### session_exercises
- session_id
- exercise_id
- order_index

### training_sets
- set_number
- weight
- reps
- is_warmup

Relational structure:

```
User
 └── Sessions
       └── Exercises
             └── Sets
```

Multi-user safe by design.

---

# Concurrency & Performance

## Multi-Worker Setup

Backend runs with multiple workers:

```
uvicorn app.api.main:app --reload --workers 4
```

This prevents blocking during:

- CSV ingestion
- Embedding computation
- Volume analytics
- Translation fallback

---

## Deployment

The full stack runs via Docker Compose:

```bash
docker compose up --build
```

| Service    | Image                  | Role                                    |
|------------|------------------------|-----------------------------------------|
| `db`       | postgres:16-alpine     | PostgreSQL database                     |
| `flyway`   | flyway/flyway:10-alpine| Schema migrations (runs before backend) |
| `backend`  | (local build)          | FastAPI app on port 8000                |
| `frontend` | (local build, Nginx)   | React SPA on port 5173 → 80             |

For horizontal scaling: Kubernetes replicas, optional Redis + task queue (Celery / RQ).

---

# Controlled Active Learning

New exercises are stored with:

- confidence score
- review requirement (if below threshold)
- persisted embedding
- canonical identity
- creator user_id

After manual validation:

- Confidence can be raised
- Exercise participates in reference embedding space
- Semantic growth remains controlled

This prevents model drift while allowing expansion.

---

# Plan Exercise Resolver

Training plan entries are first stored with `resolution_status='pending'` and can be
resolved asynchronously by the plan exercise resolver service.

Resolver behavior:

- normalizes `raw_exercise_name`
- reuses an existing exercise when possible (display/canonical match)
- creates a fallback exercise row when none exists
- sets `training_plan_exercises.exercise_id`
- sets status to `resolved` (or `failed` for invalid/unresolvable names)

Idempotency guarantees:

- only rows with `exercise_id IS NULL` and `resolution_status='pending'` are processed
- already-resolved rows are skipped on re-runs
- existing exercise rows are reused before creating new ones

Trigger strategies:

1. **Fire-and-forget after plan create/update** (default in service layer).
2. **Periodic worker/cron** via:

```bash
cd backend && python -m cli.plan_exercise_resolver_worker --interval-seconds 30
```

For one-shot execution (e.g. cron):

```bash
cd backend && python -m cli.plan_exercise_resolver_worker --limit 500
```

---

# Tech Stack

## Backend

- Python 3.11+
- FastAPI 0.134
- Uvicorn
- Pydantic v2
- PostgreSQL 16
- psycopg2 (connection pooling)
- Flyway (schema migrations)
- Docker / Docker Compose
- Argon2 password hashing
- JWT (python-jose)
- Refresh tokens (SHA-256 hashed)
- Resend (transactional email)
- Slowapi (rate limiting)
- Cryptography (token hashing)
- NumPy
- Requests
- Ollama (local LLM inference)

## Frontend

- React 19
- Vite 7
- TypeScript ~5.9 (strict mode)
- Redux Toolkit
- TanStack React Query v5
- React Router v7
- Axios (JWT auto-injection)
- Recharts
- TailwindCSS v4
- Framer Motion (animations)
- i18next + react-i18next (German + English)
- Workbox (PWA / Service Worker)
- Capacitor Android 7 (native mobile)
- zxcvbn (password strength analysis)
- Custom Toast System (Redux-driven)
- JWT-based role-aware routing (AdminRoute / ProtectedRoute)
- Vitest + MSW (testing)

## LLM Models

- `embeddinggemma` (embeddings)
- `rinex20/translategemma3:12b` (translation)

---

# Installation

## 1️⃣ Clone Repository

```bash
git clone https://github.com/Radok527/fitness-coaching-engine.git
cd fitness-coaching-engine
```

---

## 2️⃣ Install Ollama

Download: https://ollama.com/

Pull required models:

```bash
ollama pull embeddinggemma
ollama pull rinex20/translategemma3:12b
```

---

## 3️⃣ Configure .env

Copy and adjust the values:

```bash
cp .env .env.local  # or edit .env directly
```

| Variable | Description |
|---|---|
| `POSTGRES_HOST` | DB host (`db` in Docker, `localhost` local) |
| `POSTGRES_PORT` | Default `5432` |
| `POSTGRES_DB` | Database name |
| `POSTGRES_USER` | DB user |
| `POSTGRES_PASSWORD` | **Change for production** |
| `DB_POOL_MIN` / `DB_POOL_MAX` | Connection pool size (default 2/10) |
| `JWT_SECRET_KEY` | **Change for production** |
| `ALGORITHM` | `HS256` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Token lifetime |
| `OLLAMA_BASE_URL` | See note below |
| `EXTRA_CORS_ORIGINS` | Additional CORS origins (comma-separated) |

**OLLAMA_BASE_URL** depends on your setup:
- Local dev (no Docker): `http://localhost:11434`
- Docker on Windows/Mac: `http://host.docker.internal:11434`
- Docker on Linux: `http://host-gateway:11434`

---

## 4️⃣ Option A: Docker (recommended)

```bash
docker compose up --build
```

- Frontend: http://localhost:5173
- Backend API / Swagger: http://localhost:8000/docs

Schema migrations run automatically via Flyway before the backend starts.

---

## 5️⃣ Option B: Local Development

```bash
# Backend
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # macOS / Linux
pip install -r requirements.txt

# Frontend
cd frontend && npm install
```

Set `POSTGRES_HOST=localhost` in `.env` and point to a local PostgreSQL instance.

```bash
npm run dev          # Backend + Frontend concurrently
npm run dev:backend  # Backend only (4 workers)
npm run dev:reload   # Backend only (auto-reload)
npm run dev:frontend # Frontend only
npm run dev:setup    # Init DB + seed canonical exercises
```

- Backend API / Swagger: http://localhost:8000/docs
- Frontend: http://localhost:5173

---

# Roadmap

## Phase 1 — Data Foundation ✅

- [x] Canonical ontology
- [x] Hybrid classification (embedding + rule-based)
- [x] Translation fallback
- [x] Token-overlap validation guard
- [x] Confidence scoring & review system
- [x] Structured relational training model
- [x] CSV importer
- [x] Controlled semantic reference expansion

---

## Phase 2 — Analytics Engine ✅

- [x] Rolling volume aggregation
- [x] Push / Pull / Lower balance
- [x] Movement pattern tracking
- [x] Estimated 1RM tracking
- [x] Deterministic metric isolation per user

---

## Phase 3 — Progression Modeling ✅

- [x] Linear trend analysis per exercise
- [x] Plateau detection
- [x] Relative strength modeling
- [x] Performance direction classification

---

## Phase 4 — User Context Layer ✅

- [x] Multi-user architecture
- [x] JWT authentication
- [x] User profile model (age, height, experience)
- [x] Bodyweight tracking (historical)
- [x] Diet phase tracking
- [x] Goal metadata storage
- [x] Admin permission hardening

---

## Phase 5 — Training Plan & Live Execution Engine (Next)

- [x] User-defined training plans
- [x] Plan templates (warmup/work sets, reps, rest, supersets)
- [x] Plan editing (add/remove exercises & sets)
- [x] Start plan → session instantiation
- [x] Live set tracking (execution order & timestamps)
- [x] Add sets during training
- [x] Plan ≠ Session architecture enforcement

---

## Phase 6 — Goal-Oriented Coaching ✅

- [x] Conservative adaptive progression suggestion
- [x] Goal definition
- [x] Progress-to-goal tracking
- [x] Target projection engine
- [x] Goal-aware adaptive load refinement
- [x] Goal coaching service (personalized recommendations)

---

## Phase 7 — AI Coaching Layer (in progress)

- [x] Goal-aware coaching recommendations (`goal_coaching_service`)
- [x] Adaptive load progression engine (`recommendation.py`)
- [ ] LLM-based training summaries
- [ ] Structured performance interpretation
- [ ] Natural language goal planning
- [ ] Intelligent workout review

# Current Status

The system currently provides:

- JWT-based authentication with refresh tokens
- Email verification and password reset flows
- Rate limiting (Slowapi)
- Multi-user support with admin roles
- Deterministic hybrid exercise normalization
- Controlled semantic expansion with async translation queue
- Structured relational workout persistence
- Reproducible strength analytics (volume, balance, progression, plateau, relative strength / Wilks)
- Adaptive load progression recommendations
- Goal-oriented coaching with projection engine
- Training plan builder + live session execution with rest timer
- Multi-worker backend concurrency
- PostgreSQL 16 with connection pooling (psycopg2), 17 Flyway migrations
- Docker Compose deployment (db + flyway + backend + frontend)
- Progressive Web App (PWA) with Workbox offline support
- Internationalization (German + English via i18next)
- Mobile-ready (Capacitor Android 7)
- Scalable AI-ready architecture

The analytics core remains fully deterministic.

AI acts as a semantic, interpretative, and coaching assistant —
never as a computational dependency.

---

# Vision

The long-term goal is an adaptive, goal-oriented performance engine capable of:

- Forecasting strength progression
- Detecting plateaus
- Suggesting intelligent volume adjustments
- Integrating contextual signals
- Providing structured AI-assisted coaching
- Scaling horizontally via container orchestration

The architecture guarantees:

- Deterministic computation
- Controlled semantic growth
- Strict separation of concerns
- Multi-user scalability
- Infrastructure readiness for distributed deployment
