<<<<<<< HEAD
# AI-SDLC Templates

Copy these into your project's `docs/` folder as needed. Start with the phase you're in — you don't need all of them on day one.

## Quick Start

```bash
# In your project root:
mkdir docs
cp templates/3-development/AGENTS.md docs/
cp templates/3-development/CONVENTIONS.md docs/
# Fill them in ↓
```

## Template Index

```
templates/
│
├── 📘 AI-SDLC.md                 ← Master methodology guide (start here)
│
├── 📂 1_REQUIREMENT/            ← What to build
│   └── PRD.md                    ← Problem, user stories, metrics, scope
│
├── 📂 2_DESIGN/                  ← How to build it
│   ├── DESIGN_SPEC.md            ← Technical design, data flow, API contract
│   └── ARCHITECTURE.md           ← Design decisions, patterns, tech debt
│
├── 📂 3_DEVELOPMENT/             ← Build it
│   ├── AGENTS.md                 ← Project identity, stack, commands ★
│   ├── CONVENTIONS.md            ← Code style, naming, git, PR rules
│   ├── DATABASE.md               ← Schema, queries, migrations
│   ├── API_PATTERNS.md           ← REST conventions, response shapes
│   └── GLOSSARY.md               ← Domain terms, state machines
│
├── 📂 4_TESTING/                 ← Verify it
│   ├── TESTING.md                ← Test pyramid, patterns, coverage
│   └── SECURITY.md               ← Auth rules, attack defense, hard constraints
│
├── 📂 5_DEPLOYMENT/              ← Ship it
│   ├── DEPLOYMENT.md             ← Environments, CI/CD, rollback
│   └── DEVELOPMENT.md            ← Local setup, env vars, troubleshooting
│
└── 📂 6_MAINTENANCE/             ← Keep it alive
    ├── RUNBOOK.md                ← Ops commands, incident response
    ├── MONITORING.md             ← Dashboards, metrics, alerts
    ├── CHANGELOG.md              ← Release history, migration guides
    └── TASKS.md                  ← Active work, backlog, tech debt
```

★ = absolute minimum. Start here.

## Per-Phase READMEs

<<<<<<< HEAD
Each phase folder contains a README.md with:
- What this phase covers
- When to use each template
- Example prompts for AI agents
- Deliverables checklist
=======
---

## How Agents Use These

The pattern is:

1. **AGENTS.md** loads every session — keep it tight (~1-2 KB)
2. **AGENTS.md links to others** — agent loads them on-demand when relevant
3. **Example** from AGENTS.md:
   ```markdown
   ## Further Context
   - Database patterns: [DATABASE.md](./DATABASE.md)
   - API conventions: [API_PATTERNS.md](./API_PATTERNS.md)
   - Security rules: [SECURITY.md](./SECURITY.md) (read before any auth/payment code)
   ```

The agent reads AGENTS.md → sees you're working on payments → loads SECURITY.md + DATABASE.md → writes code that follows both.

---

When you're ready to spin up that personal project, tell me the directory and I'll read your stack and fill these out for real. 🔮
>>>>>>> 52fb6bc21c0edc3bc95df97a3137312e53741a07
