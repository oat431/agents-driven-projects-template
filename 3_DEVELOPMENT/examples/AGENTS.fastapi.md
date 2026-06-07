# AGENTS.md — short-link (FastAPI + PostgreSQL)

<!-- Real example. Python 3.12 + FastAPI + SQLAlchemy + Redis. -->

## Project

- **Name:** short-link
- **Purpose:** URL shortener with click analytics. REST API only — no frontend.
- **Repo:** github.com/panomete/short-link
- **URL:** <https://s.panomete.com>

## Stack

- **Language:** Python 3.12
- **Framework:** FastAPI 0.110
- **Database:** PostgreSQL 16 (db_short_link)
- **Cache:** Redis 7 (redirect cache, rate limiting)
- **ORM:** SQLAlchemy 2.0 (async) + Alembic
- **Validation:** Pydantic v2
- **Testing:** pytest + pytest-asyncio + httpx
- **Lint:** Ruff (format + lint)
- **Container:** Docker Compose

## Project Map

```
app/
├── main.py                       — FastAPI app, lifespan, CORS
├── config.py                     — Settings (pydantic-settings, from .env)
├── api/
│   └── v1/
│       ├── router.py             — /api/v1/*
│       ├── links.py              — POST /shorten, GET /{slug}, DELETE /{slug}
│       ├── analytics.py          — GET /{slug}/stats
│       └── auth.py               — POST /api-key (for API access)
├── models/
│   ├── link.py                   — Link ORM model
│   └── click.py                  — Click ORM model
├── schemas/
│   ├── link.py                   — Pydantic request/response
│   └── click.py                  — Click analytics response
├── services/
│   ├── shortener.py              — Slug generation, collision handling
│   └── analytics.py              — Click aggregation
├── core/
│   ├── database.py               — AsyncSession, get_db dependency
│   ├── redis.py                  — Redis client, cache helpers
│   └── security.py               — API key validation
└── migrations/                    — Alembic

tests/
├── conftest.py                   — Fixtures (test DB, async client)
├── test_links.py                 — CRUD + redirect tests
├── test_analytics.py             — Click tracking tests
└── test_shortener.py             — Slug generation edge cases
```

## Commands

```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run (needs PostgreSQL + Redis)
docker compose up -d
uvicorn app.main:app --reload --port 8000
# → http://localhost:8000
# → Swagger: http://localhost:8000/docs

# Migrations
alembic upgrade head             # Apply
alembic revision --autogenerate -m "add clicks table"  # Create
alembic downgrade -1             # Rollback (dev only)

# Test
pytest                           # All tests
pytest tests/test_links.py -v   # Specific file
pytest --cov=app --cov-report=html

# Lint
ruff check .
ruff format .

# Type check
mypy app/
```

## Conventions

- Python 3.12+. Type hints on EVERY function signature (strict mode).
- Async everywhere: `async def` + `AsyncSession` + `httpx.AsyncClient`.
- Pydantic v2 for all request/response schemas. `model_validate()` not `parse_obj()`.
- Services never import FastAPI dependencies — they receive plain values.
- SQLAlchemy 2.0 style: `select(Link).where(Link.slug == slug)` — no `Query` object.
- Database sessions via `async with get_db() as db:`.
- Error handling: raise `HTTPException` in route layer, custom exceptions in services.
- Slug generation: `secrets.token_urlsafe(6)` — 6 chars, retry on collision.
- Redis keys: `redirect:{slug}` (URL cache), `rate:{ip}` (rate limit).

## Security Rules

- API key in `Authorization: Bearer <key>` header. Validated via dependency.
- Rate limit: 30 POST /shorten per minute per IP (Redis).
- No dangerous redirects: validate target URL doesn't redirect to itself.
- SQL injection impossible via SQLAlchemy parameterized queries — still be careful with raw SQL.
- `.env` gitignored. `SECRET_KEY` for API key hashing.

## Constraints

- Do NOT change slug generation algorithm (affects existing links).
- Shortened URLs: never delete — soft-deactivate only (`is_active = false`).
- Click tracking: async (fire-and-forget to not slow redirect).
- API versioned at `/api/v1/`. Breaking changes → `/api/v2/`.

## Key Files

- `app/config.py` — All settings. Read first.
- `app/core/database.py` — DB session setup.
- `app/models/link.py` — Central data model.
- `tests/conftest.py` — Test fixtures and helpers.
