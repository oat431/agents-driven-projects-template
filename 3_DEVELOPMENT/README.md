# Phase 3: Development

**What:** Write working, tested, reviewable code that follows conventions.  
**Who:** AI agent implements. Human reviews and approves.  
**When:** After design approved. This is the main loop.

---

## Template Index

```
3_DEVELOPMENT/
│
├── 📄 README.md              ← This file
├── 📄 AGENTS.md              ← ★ Project identity (template)
├── 📄 CONVENTIONS.md         ← Code style, naming, git
├── 📄 DATABASE.md            ← Schema, queries, migrations
├── 📄 API_PATTERNS.md        ← REST endpoint conventions
├── 📄 GLOSSARY.md            ← Domain terms
│
└── 📂 examples/              ← Real project folder examples
    │
    ├── 📂 project-api/       ← Spring Boot REST API
    │   ├── AGENTS.md         ← Java 21, PostgreSQL, Redis
    │   ├── AGENTS.fastapi.md ← Alternative: Python FastAPI
    │   └── API_PATTERNS.md   ← Controller template, DTO, pagination
    │
    ├── 📂 project-web/       ← Next.js Admin Dashboard
    │   ├── AGENTS.md         ← TypeScript, shadcn/ui, TanStack Query
    │   ├── AGENTS.nuxt3.md   ← Alternative: Vue 3 / Nuxt 3
    │   ├── UI_SPEC.md        ← Colors, fonts, spacing, components
    │   └── COMPONENT_TREE.md ← ui/ → layout/ → data/ → feature/
    │
    ├── 📂 project-fullstack/ ← Monorepo: API + Web
    │   ├── AGENTS.md         ← Shared types, multi-package
    │   └── API_CONTRACT.md   ← Shared truth between sides
    │
    ├── 📂 project-batch/     ← Background Job Processor
    │   └── AGENTS.md         ← Spring Batch, RabbitMQ, idempotency
    │
    └── 📂 project-infra/     ← Docker Compose Homelab
        ├── AGENTS.md         ← Multi-service, nginx, monitoring
        └── DEPLOYMENT_WORKFLOW.md ← Env chain, cron, rollback
```

---

## How to Use the Examples

Each `project-*/` folder shows what a real project's `docs/` directory looks like — not isolated templates, but files that work together.

```
My real project:
docs/
├── AGENTS.md            ← Copy from closest example, tweak
├── API_PATTERNS.md      ← Copy from project-api/ example
└── ...                  ← Add as needed
```

**Pick your closest match:**

| Your Project | Copy From |
|-------------|-----------|
| REST API (Java) | `project-api/` |
| REST API (Python) | `project-api/AGENTS.fastapi.md` |
| React admin dashboard | `project-web/` |
| Vue storefront | `project-web/AGENTS.nuxt3.md` |
| Full-stack monorepo | `project-fullstack/` |
| Background worker | `project-batch/` |
| Docker/infra setup | `project-infra/` |

---

## Workflow

1. Agent reads DESIGN_SPEC.md → AGENTS.md → CONVENTIONS.md → DATABASE.md
2. Agent implements: migrations → entities → services → controllers → tests
3. Agent self-reviews: runs tests, lint, checkstyle
4. Agent opens PR with description
5. You review → feedback → agent updates → merge
6. Repeat for next feature

## Deliverables

- [ ] Feature implemented per design
- [ ] Tests pass (unit + integration)
- [ ] Lint/checkstyle clean
- [ ] PR opened with clear description
- [ ] CHANGELOG.md entry added (Unreleased section)
