# AI-SDLC Templates

Copy these into your project's `docs/` folder as needed. Start with the phase you're in.

## Quick Start (30 Seconds)

```bash
# In your new project:
mkdir docs
cp templates/3_DEVELOPMENT/AGENTS.md docs/
cp templates/3_DEVELOPMENT/CONVENTIONS.md docs/
# Fill them in. Done.
```

→ See [QUICKSTART.md](./QUICKSTART.md) for the 5-minute walkthrough.

---

## Full Template Index

```
templates/
│
├── 📘 AI-SDLC.md                     ← Master methodology (read this first)
├── 📘 QUICKSTART.md                  ← 5-minute onboarding
├── 📘 PROJECT_CHECKLIST.md           ← New project bootstrap checklist
│
├── 📂 1_REQUIREMENT/                 ← What to build
│   ├── README.md
│   ├── PRD.md                        ← Lightweight: problem, stories, metrics
│   └── SRS/                          ← Formal: IEEE 830 style (optional)
│       └── SRS_TEMPLATE.md
│
├── 📂 2_DESIGN/                       ← How to build it
│   ├── README.md
│   ├── ARCHITECTURE.md               ← System-wide decisions
│   ├── DESIGN_SPEC.md                ← Backend: API, DB, security design
│   ├── ADR.md                        ← Architecture Decision Records
│   ├── UI_SPEC.md                    ← Frontend: colors, fonts, spacing
│   ├── PAGE_SPEC.md                  ← Frontend: routes, states, data
│   ├── COMPONENT_TREE.md             ← Frontend: component hierarchy
│   ├── API_CONTRACT.md               ← Bridge: frontend ↔ backend
│   ├── HLDD/                         ← High-Level Design (optional)
│   │   └── HLDD_TEMPLATE.md
│   ├── LLDD/                         ← Low-Level Design (optional)
│   │   └── LLDD_TEMPLATE.md
│   └── DIAGRAMS/                     ← Mermaid diagram examples
│       ├── example-erd.mmd
│       ├── example-sequence.mmd
│       ├── example-deployment.mmd
│       └── example-component.mmd
│
├── 📂 3_DEVELOPMENT/                 ← Build it
│   ├── README.md
│   ├── AGENTS.md                     ← Project identity, stack, commands ★
│   ├── CONVENTIONS.md                ← Code style, naming, git
│   ├── DATABASE.md                   ← Schema, queries, migrations
│   ├── API_PATTERNS.md               ← REST conventions
│   ├── GLOSSARY.md                   ← Domain terms
│   └── examples/                     ← Filled-in AGENTS.md by stack
│       ├── AGENTS.spring-boot.md
│       ├── AGENTS.nextjs.md
│       ├── AGENTS.nuxt3.md
│       ├── AGENTS.fastapi.md
│       └── AGENTS.homelab.md
│
├── 📂 4_TESTING/                     ← Verify it
│   ├── README.md
│   ├── TEST_STRATEGY.md              ← Pyramid, coverage, quality gates
│   ├── UNIT_TEST.md                  ← Backend + Frontend patterns
│   ├── INTEGRATION_TEST.md           ← API + DB slices
│   ├── E2E_TEST.md                   ← Playwright/Cypress flows
│   ├── UAT.md                        ← Traced to SRS/PRD
│   ├── PERFORMANCE_TEST.md           ← k6 load/stress/benchmarks
│   └── SECURITY.md                   ← Security review checklist
│
├── 📂 5_DEPLOYMENT/                  ← Ship it
│   ├── README.md
│   ├── DEPLOYMENT_WORKFLOW.md        ← Multi-env pipeline + promotion gates
│   ├── CI_CD.md                      ← GitHub Actions pipeline design
│   ├── DEPLOYMENT.md                 ← Infrastructure + commands
│   └── DEVELOPMENT.md                ← Local setup + troubleshooting
│
└── 📂 6_MAINTENANCE/                 ← Keep it alive
    ├── README.md
    ├── RUNBOOK.md                    ← Ops commands, incident response
    ├── MONITORING.md                 ← Dashboards, metrics, alerts
    ├── CHANGELOG.md                  ← Release history
    └── TASKS.md                      ← Backlog, bugs, tech debt
```

★ = absolute minimum. Start here.

---

## Per-Phase READMEs

Each phase folder contains a README.md with:
- What this phase covers
- When to use each template
- Example prompts for AI agents
- Deliverables checklist

---

## How Agents Use These

1. **AGENTS.md** loads every session
2. **AGENTS.md links to others** — agent loads on-demand
3. **Example link block** from AGENTS.md:

```markdown
## Further Context
- Database patterns: [DATABASE.md](./DATABASE.md)
- API conventions: [API_PATTERNS.md](./API_PATTERNS.md)
- Security rules: [SECURITY.md](./SECURITY.md)
- Architecture decisions: [ADR.md](./ADR.md)
```

The agent reads AGENTS.md → sees "payment" task → loads SECURITY.md + DATABASE.md → writes correct code.
