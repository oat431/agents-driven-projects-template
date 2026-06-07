# AGENTS.md — project-fullstack (Monorepo: API + Web)

## Project
- **Name:** fullstack-app
- **Purpose:** E-commerce platform. Monorepo with shared types, API server, and admin dashboard.
- **Repo:** github.com/panomete/fullstack-app

## Repo Structure
```
fullstack-app/
├── packages/
│   ├── shared/                   ← Shared types, Zod schemas, constants
│   ├── api/                      ← Spring Boot REST API (project-api)
│   └── web/                      ← Next.js admin dashboard (project-web)
├── docker-compose.yml            ← Local dev: PostgreSQL, Redis, API, Web
├── docs/                         ← AI-SDLC docs
└── README.md
```

## When Working in This Repo

**API changes:** Read `packages/api/docs/AGENTS.md` + `API_PATTERNS.md`.  
**Web changes:** Read `packages/web/docs/AGENTS.md` + `UI_SPEC.md`.  
**Shared types:** Read `packages/shared/README.md`. Both sides consume from here.  
**Contract changes:** Read `docs/API_CONTRACT.md` — update BOTH sides.

## Stack
| Package | Stack |
|---------|-------|
| `packages/shared` | TypeScript 5.x, Zod |
| `packages/api` | Java 21, Spring Boot 3.3, PostgreSQL 16 |
| `packages/web` | Next.js 14, TypeScript, Tailwind, shadcn/ui |

## Commands
```bash
# Everything (Docker Compose)
docker compose up -d

# API only
cd packages/api && ./mvnw spring-boot:run

# Web only
cd packages/web && pnpm dev

# Test all
cd packages/api && ./mvnw test
cd packages/web && pnpm test

# Generate shared types for both sides
cd packages/shared && pnpm build
```

## Key Rule: Shared Types
`packages/shared/` is the source of truth for data shapes. Both API and Web consume from it. When changing a type:
1. Update in `packages/shared/src/`
2. Run `pnpm build` in shared
3. Update API DTOs + Web components to match
4. Update `API_CONTRACT.md` if the contract changed
