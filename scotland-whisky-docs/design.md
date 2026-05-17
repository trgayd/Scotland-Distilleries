# Design Document
## Scotland Whisky Platform

Version: 0.1 (Draft)
Date: 2026-05-16

## 1. System Overview

Three-tier web application:

React Frontend (Vite + TypeScript + TailwindCSS)
communicates via HTTP REST to
FastAPI Backend (Python, async, JWT auth)
which connects to
PostgreSQL Database (Docker locally / RDS on AWS)

All three tiers run in Docker locally. On AWS, each tier maps directly to a managed service with no code changes - only environment config changes.

## 2. Technology Stack

| Layer | Local | AWS Equivalent |
|-------|-------|----------------|
| Frontend | Vite + React + TypeScript | S3 + CloudFront |
| Backend | FastAPI (Python 3.12) | Lambda + API Gateway or ECS Fargate |
| Database | PostgreSQL 16 (Docker) | RDS PostgreSQL |
| Auth | JWT (python-jose) | Same (stateless) |
| ORM | SQLAlchemy 2.0 (async) | Same |
| Migrations | Alembic | Same |
| Container orchestration | Docker Compose | ECS / CDK |
| AI enrichment | Claude API (Anthropic SDK) | Same (Lambda job) |

## 3. Local Development Environment

### Docker Compose Services

db - PostgreSQL 16
backend - FastAPI (uvicorn, hot reload)
frontend - Vite dev server (hot reload)

### Port Mapping

| Service | Local Port |
|---------|------------|
| Frontend | 5173 |
| Backend API | 8000 |
| PostgreSQL | 5432 |

### Environment Variables (.env, not committed)

DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/scotchdb
SECRET_KEY=...
ANTHROPIC_API_KEY=...
ENVIRONMENT=local

## 4. Database Schema

### Entity Relationships

distilleries - has many - whiskies
distilleries - many to many - users (via user_distilleries)
whiskies - many to many - users (via user_whiskies)
whiskies - has many - whisky_submissions (from users)

### distilleries

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| name | VARCHAR(255) | |
| region | VARCHAR(50) | Speyside / Highland / Islay / Lowland / Campbeltown / Island |
| town | VARCHAR(255) | |
| founded_year | INTEGER | |
| latitude | DECIMAL(9,6) | |
| longitude | DECIMAL(9,6) | |
| description | TEXT | |
| production_style | VARCHAR(50) | peated / unpeated / lightly_peated |
| is_active | BOOLEAN | still operating |
| data_source | VARCHAR(50) | manual / ai_generated / user_submitted |
| is_published | BOOLEAN | |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### distillery_expressions

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| distillery_id | UUID FK | |
| name | VARCHAR(255) | e.g. 16 Year Old |

### whiskies

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| distillery_id | UUID FK | |
| name | VARCHAR(255) | |
| age_years | INTEGER | nullable (NAS) |
| abv | DECIMAL(4,1) | |
| cask_type | VARCHAR(255) | |
| flavour_profile | TEXT | |
| tasting_notes | TEXT | |
| is_limited | BOOLEAN | |
| data_source | VARCHAR(50) | manual / ai_generated / user_submitted |
| is_published | BOOLEAN | |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### users

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| email | VARCHAR(255) UNIQUE | |
| password_hash | VARCHAR(255) | bcrypt |
| display_name | VARCHAR(100) | |
| role | VARCHAR(20) | user / moderator / admin |
| is_active | BOOLEAN | |
| created_at | TIMESTAMPTZ | |

### user_distilleries

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK | |
| distillery_id | UUID FK | |
| liked | BOOLEAN | |
| visited | BOOLEAN | |
| visited_at | DATE | nullable |
| want_to_visit | BOOLEAN | |
| notes | TEXT | nullable |
| created_at | TIMESTAMPTZ | |

UNIQUE constraint: (user_id, distillery_id)

### user_whiskies

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK | |
| whisky_id | UUID FK | |
| tasted | BOOLEAN | |
| tasted_at | DATE | nullable |
| rating | VARCHAR(20) | love / like / neutral / dislike - nullable |
| want_to_taste | BOOLEAN | |
| want_to_buy | BOOLEAN | |
| owned | BOOLEAN | |
| tasting_notes | TEXT | nullable |
| created_at | TIMESTAMPTZ | |

UNIQUE constraint: (user_id, whisky_id)

### whisky_submissions

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| submitted_by | UUID FK -> users | |
| distillery_id | UUID FK | nullable |
| name | VARCHAR(255) | |
| age_years | INTEGER | nullable |
| abv | DECIMAL(4,1) | nullable |
| cask_type | VARCHAR(255) | nullable |
| flavour_profile | TEXT | nullable |
| status | VARCHAR(20) | pending / approved / rejected |
| reviewed_by | UUID FK -> users | nullable |
| reviewed_at | TIMESTAMPTZ | nullable |
| reviewer_notes | TEXT | nullable |
| created_at | TIMESTAMPTZ | |

### ai_enrichment_logs

| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| job_type | VARCHAR(50) | distillery_update / whisky_add |
| target_id | UUID | distillery or whisky id |
| fields_updated | JSONB | |
| triggered_by | UUID FK -> users | admin user |
| status | VARCHAR(20) | pending / reviewed / applied / rejected |
| created_at | TIMESTAMPTZ | |

## 5. API Design

Base URL: /api/v1

### Auth
POST /auth/register
POST /auth/login
POST /auth/logout
GET /auth/me

### Distilleries
GET /distilleries - list, filter, search, sort
GET /distilleries/{id} - detail
GET /distilleries/{id}/whiskies - whiskies for a distillery

### Whiskies
GET /whiskies - list, filter, search, sort
GET /whiskies/{id} - detail
POST /whiskies/submit - user submission (auth required)

### User Interactions
GET /me/distilleries - user's distillery interactions
PUT /me/distilleries/{id} - update liked / visited / want_to_visit
GET /me/whiskies - user's whisky interactions
PUT /me/whiskies/{id} - update tasted / rating / want_to_buy etc.

### Admin
GET /admin/submissions - moderation queue
PUT /admin/submissions/{id} - approve / reject
POST /admin/enrich/distilleries - trigger AI enrichment
GET /admin/enrichment-logs - view enrichment history

All responses: { "data": { ... }, "meta": { "total": 100, "page": 1, "per_page": 20 } }

## 6. Frontend Structure

src/
  components/ (distilleries, whiskies, user, admin, shared)
  pages/ (Home, Distilleries, DistilleryDetail, Whiskies, WhiskyDetail, Dashboard, Admin)
  api/ - typed API client
  hooks/ - React Query hooks
  store/ - auth state (Zustand)
  types/ - shared TypeScript types

Key libraries: React Query, Zustand, React Router v6, TailwindCSS

## 7. Authentication Flow

1. User POSTs credentials to /auth/login
2. Backend returns access_token (JWT, 1 hour) + refresh_token (7 days)
3. Frontend stores access_token in memory, refresh_token in httpOnly cookie
4. All authenticated requests send Authorization: Bearer access_token
5. On expiry, frontend uses refresh token to get a new access token

Mobile-compatible - the mobile app will use the same token flow.

## 8. AI Enrichment Design

- Standalone Python script / FastAPI background task
- Uses Claude API with structured output (JSON mode)
- Targets specific records, not bulk sweeps
- Output written to ai_enrichment_logs with status pending
- Admin reviews in a diff UI before applying
- Applied changes set data_source = ai_generated on the record

## 9. AWS Deployment Architecture (Future)

Route 53
  CloudFront
    S3 (React build)
    API Gateway
      Lambda (FastAPI via Mangum) or ECS Fargate
        RDS PostgreSQL (private subnet)
          Secrets Manager (env vars)

CDK for all infrastructure as code. Separate stacks for networking, database, backend, frontend. CI/CD via GitHub Actions: test, build, deploy.

## 10. Project Folder Structure

scotland-whisky/
  backend/
    app/
      api/ - route handlers
      models/ - SQLAlchemy models
      schemas/ - Pydantic schemas
      services/ - business logic
      db/ - session, Alembic migrations
      core/ - config, auth, security
    tests/
    Dockerfile
    requirements.txt
  frontend/
    src/
    public/
    Dockerfile
    package.json
  docs/
    requirements.md
    design.md
  docker-compose.yml
  .env.example
  CLAUDE.md
