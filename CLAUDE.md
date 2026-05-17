# CLAUDE.md — Scotland Whisky Platform

This file provides context for Claude Code sessions working on the `scotland-whisky` project.

## What This Project Is

A full-stack web platform for exploring Scottish whisky distilleries and whiskies. Users can discover distilleries and whiskies, log visits and tastings, rate whiskies, and build wishlists. The platform is data-driven, with records maintainable and enrichable via AI.

A future mobile app is planned. The backend is designed from the start to serve both web and mobile.

## Companion Repo

- `trgayd/scotland-distilleries` — the original prototype (single `index.html`, React via CDN, hardcoded data). Use `index.html` as a visual/design reference — it establishes the amber/stone colour palette, card layout, region filtering, and heritage sort UX.
- `trgayd/scotland-whisky` — this repo, the proper full-stack implementation.

## Key Documents

- `docs/requirements.md` — full functional and non-functional requirements, user personas, use cases
- `docs/design.md` — architecture, database schema, API design, frontend structure, auth flow, AWS deployment plan

Read both before making any significant changes.

## Stack

| Layer | Technology |
|-------|------------|
| Frontend | React + Vite + TypeScript + TailwindCSS |
| Backend | FastAPI (Python 3.12, async) |
| Database | PostgreSQL 16 |
| ORM | SQLAlchemy 2.0 (async) + Alembic migrations |
| Auth | JWT (access token 1h + refresh token 7d, httpOnly cookie) |
| AI enrichment | Anthropic Claude API |
| Local orchestration | Docker Compose |
| Future cloud | AWS (S3+CloudFront, Lambda/ECS, RDS, CDK) |

## Local Development

```bash
docker-compose up
```

- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- PostgreSQL: localhost:5432

Copy `.env.example` to `.env` and fill in values before running.

## Project Structure

```
scotland-whisky/
  backend/
    app/
      api/       # route handlers
      models/    # SQLAlchemy models
      schemas/   # Pydantic schemas
      services/  # business logic
      db/        # session, Alembic migrations
      core/      # config, auth, security
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
```

## Database (Summary)

Key tables: `distilleries`, `distillery_expressions`, `whiskies`, `users`, `user_distilleries`, `user_whiskies`, `whisky_submissions`, `ai_enrichment_logs`.

All records have `data_source` (manual / ai_generated / user_submitted) and `is_published` flags. Soft deletes only.

See `docs/design.md` section 4 for full schema.

## API (Summary)

Base: `/api/v1`

- `POST /auth/register`, `POST /auth/login`, `GET /auth/me`
- `GET /distilleries`, `GET /distilleries/{id}`, `GET /distilleries/{id}/whiskies`
- `GET /whiskies`, `GET /whiskies/{id}`, `POST /whiskies/submit`
- `GET/PUT /me/distilleries`, `GET/PUT /me/whiskies`
- `GET/PUT /admin/submissions`, `POST /admin/enrich/distilleries`

See `docs/design.md` section 5 for full API design.

## Design Language

Continue the visual style from `trgayd/scotland-distilleries/index.html`:
- Colours: amber (primary), stone (neutral backgrounds and text)
- Typography: Inter (body), Lora (headings)
- Components: rounded cards with subtle shadows, amber hover/active states

## Current Status (as of 2026-05-17)

- Requirements and design documents complete (see `docs/`)
- Project scaffolding: not yet started
- Next step: scaffold Docker Compose + FastAPI backend + Vite/React frontend, seed distillery data from `scotland-distilleries/index.html` into PostgreSQL

## What To Do Next

1. Scaffold `docker-compose.yml` with db, backend, frontend services
2. Set up FastAPI app skeleton with SQLAlchemy models matching the schema in `docs/design.md`
3. Set up Alembic migrations
4. Create seed script to import the ~75 distilleries from `scotland-distilleries/index.html` into the database
5. Implement distillery list and detail API endpoints
6. Set up Vite + React frontend, wire up to API
