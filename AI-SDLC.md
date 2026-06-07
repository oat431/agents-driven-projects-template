# AI-SDLC.md — AI-Assisted Software Development Life Cycle

## What Is AI-SDLC?

Traditional SDLC: humans do everything. AI-SDLC: humans decide, AI executes. You stay in the driver's seat — the agent reads your context files and does the heavy lifting.

```
                    ┌── YOU ──┐
                    │ Decide   │
                    │ Review   │
                    │ Approve  │
                    └────┬─────┘
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    ▼                    ▼                    ▼
┌───────┐          ┌───────────┐        ┌──────────┐
│ AGENT │  reads   │  CONTEXT  │ writes │  AGENT   │
│       │──────────▶  FILES    │◀────────│          │
└───────┘          └───────────┘        └──────────┘
```

---

## Phase 0: Project Bootstrap

**Goal:** Initialize the project with enough context that any AI agent can be productive immediately.

### What You Do

1. Create the project directory
2. Initialize git, package manager, framework scaffold
3. Copy the relevant templates from this guide into `docs/`
4. Fill in `AGENTS.md` — this is non-negotiable

### What The Agent Does

Nothing yet. This is setup.

### Deliverables

- [ ] `AGENTS.md` filled in (stack, commands, conventions)
- [ ] Git repo initialized
- [ ] Framework scaffolded

---

## Phase 1: Requirements

**Goal:** Define what to build before building it. The agent validates implementation against this.

### Context Files

| File | Purpose | Level |
|------|---------|-------|
| **PRD.md** | Problem, user stories, success metrics, scope | Must |
| SRS/SRS_TEMPLATE.md | Formal IEEE 830 spec (for compliance/multi-module) | Optional |
| GLOSSARY.md | Domain terms if the domain is complex | Nice |

### Workflow

```
YOU: "I want to build a link shortener with analytics"
  │
  ▼
YOU write PRD.md (or ask agent to draft it from your description)
  │
  ▼
AGENT reads PRD.md
  │
  ▼
AGENT asks clarifying questions:
  - "Should shortened URLs expire?"
  - "Do you need custom slugs or only auto-generated?"
  - "What analytics? Click count, geo, referrer?"
  │
  ▼
YOU answer → AGENT updates PRD.md
  │
  ▼
PRD.md is approved → move to Design
```

### Example Prompt

> "Read my PRD draft below. Ask clarifying questions. Identify edge cases I missed. Propose success metrics."

### Deliverables

- [ ] `PRD.md` approved
- [ ] Edge cases documented
- [ ] Success metrics defined
- [ ] Scope boundaries explicit (what's NOT included)

---

## Phase 2: Design

**Goal:** Decide HOW to build before coding. Prevents architecture regret.

### Context Files

**System (both sides):**
| File | Purpose | Level |
|------|---------|-------|
| **ARCHITECTURE.md** | System-wide decisions, patterns, tech debt | Must |
| ADR.md | Architecture Decision Records (why we chose X) | High |
| DIAGRAMS/ | Mermaid ERD, sequence, deployment, component | High |

**Backend:**
| File | Purpose | Level |
|------|---------|-------|
| **DESIGN_SPEC.md** | Technical design, data flow, API contract | Must |
| HLDD/HLDD_TEMPLATE.md | High-Level Design: module decomposition | If multi-module |
| LLDD/LLDD_TEMPLATE.md | Low-Level Design: class detail, algorithms | If complex module |

**Frontend:**
| File | Purpose | Level |
|------|---------|-------|
| UI_SPEC.md | Design tokens: colors, fonts, spacing, radius | If UI exists |
| PAGE_SPEC.md | Routes + per-page: data, states, interactions | If UI exists |
| COMPONENT_TREE.md | Component hierarchy: ui/ → layout/ → feature/ | If UI exists |
| API_CONTRACT.md | Bridge: what frontend expects from backend | If UI + API |

### Workflow

```
YOU: "Here's the PRD. Design the system."
  │
  ▼
AGENT reads PRD.md
  │
  ▼
AGENT drafts DESIGN_SPEC.md:
  - Component diagram
  - Data flow for each user story
  - API contract (request/response shapes)
  - Data model (new tables, columns, relations)
  - Security considerations
  - Rollout plan
  │
  ▼
YOU review & challenge:
  - "Why REST over GraphQL?"
  - "Should we use Redis or just the DB for rate limiting?"
  │
  ▼
AGENT updates DESIGN_SPEC.md with decisions + rationale
  │
  ▼
DESIGN_SPEC.md approved → move to Development
```

### Example Prompt

> "Read PRD.md. Design the system. Include: component diagram, data flow per user story, API contract, database schema changes, and a rollout plan. Flag any security concerns."

### Deliverables

- [ ] `DESIGN_SPEC.md` approved
- [ ] Architecture decisions documented with rationale
- [ ] API contract defined (endpoints, request/response shapes)
- [ ] Database schema designed
- [ ] `ARCHITECTURE.md` updated with key decisions

---

## Phase 3: Development

**Goal:** Write working, tested, reviewable code that follows conventions.

### Context Files

| File | Purpose | Level |
|------|---------|-------|
| **AGENTS.md** | Project identity, commands, constraints | Must |
| **CONVENTIONS.md** | Code style, naming, git conventions | Must |
| DATABASE.md | Query patterns, migration rules | If DB |
| API_PATTERNS.md | Endpoint templates, error shapes | If API |
| SECURITY.md | Hard rules the agent must never break | If auth/PII |
| GLOSSARY.md | Domain terminology | If complex domain |

### Workflow

```
YOU: "Implement the 2FA setup flow from DESIGN_SPEC.md"
  │
  ▼
AGENT reads DESIGN_SPEC.md → AGENTS.md → CONVENTIONS.md → DATABASE.md
  │
  ▼
AGENT implements:
  1. Migration file (V004__add_user_totp.sql)
  2. UserTotp entity + repository
  3. TotpService (generate, verify, recovery codes)
  4. AuthController endpoints
  5. Unit tests
  6. Integration test
  │
  ▼
AGENT runs: ./mvnw test  (fixes failures)
  │
  ▼
AGENT presents PR:
  - "Created PR #456: Add TOTP-based 2FA"
  - Summary of changes, how to test, what to review
  │
  ▼
YOU review → approve or request changes → AGENT updates → merge
```

### Development Loop

```
┌──────────────────────────────────────┐
│  1. YOU assign task                  │
│  2. AGENT reads context files        │
│  3. AGENT implements + tests         │
│  4. AGENT self-reviews (lint, test)  │
│  5. AGENT opens PR                   │
│  6. YOU review → feedback            │
│  7. AGENT updates                    │
│  8. Merge                            │
│         ↻ back to 1                  │
└──────────────────────────────────────┘
```

### Example Prompt

> "Implement US-1 from PRD.md: user enables 2FA via authenticator app. Follow the design in DESIGN_SPEC.md. Use the patterns in DATABASE.md and CONVENTIONS.md. Write unit tests + integration test. Run tests before opening PR."

### Deliverables

- [ ] Feature implemented per design spec
- [ ] Tests pass (unit + integration)
- [ ] Lint/checkstyle passes
- [ ] PR opened with clear description
- [ ] `CHANGELOG.md` entry added (under Unreleased)

---

## Phase 4: Testing & Quality

**Goal:** Verify the feature works, doesn't break existing functionality, and meets quality bars.

### Context Files

| File | Purpose |
|------|---------|
| **TEST_STRATEGY.md** | Master plan: pyramid, coverage targets, quality gates |
| UNIT_TEST.md | Backend (JUnit/pytest) + Frontend (Vitest) patterns |
| INTEGRATION_TEST.md | API + DB slices with real database |
| E2E_TEST.md | Playwright/Cypress user flow scenarios |
| UAT.md | Traced to SRS/PRD user stories with sign-off |
| PERFORMANCE_TEST.md | k6 load/stress/benchmarks + thresholds |
| SECURITY.md | Security review checklist |
| PRD.md | Verify user stories are satisfied |

### Workflow

```
YOU: "Write a test plan for the 2FA feature"
  │
  ▼
AGENT reads PRD.md → DESIGN_SPEC.md → TESTING.md
  │
  ▼
AGENT generates test cases:
  - Unit: TOTP code generation (RFC 6238 test vectors)
  - Unit: Recovery code generation (uniqueness, format)
  - Integration: Full setup → verify → login flow
  - Edge: Clock skew, rapid submissions, token replay
  - Security: Rate limiting, encryption verification
  │
  ▼
AGENT runs full test suite → reports coverage
  │
  ▼
YOU: "Add security fuzzing for the 2FA endpoint"
  │
  ▼
AGENT adds fuzz tests → reports findings
```

### Example Prompt

> "Review the 2FA implementation against the PRD. Identify missing test coverage. Add tests for edge cases: clock skew, rate limiting, recovery code exhaustion, concurrent enable/disable."

### Deliverables

- [ ] Test plan executed
- [ ] Coverage meets targets (from TESTING.md)
- [ ] Edge cases tested
- [ ] Security review passed (from SECURITY.md)
- [ ] Regression suite green

---

## Phase 5: Deployment

**Goal:** Ship to production safely with rollback capability.

### Context Files

| File | Purpose |
|------|---------|
| **DEPLOYMENT_WORKFLOW.md** | Multi-env pipeline, promotion gates, rollback |
| CI_CD.md | GitHub Actions / GitLab CI pipeline design |
| DEPLOYMENT.md | Infrastructure + deployment commands |
| DEVELOPMENT.md | Local setup + troubleshooting |
| SECURITY.md | Secrets handling, production constraints |
| CHANGELOG.md | Release notes |

### Workflow

```
YOU: "Deploy 2FA feature to staging"
  │
  ▼
AGENT reads DEPLOYMENT.md → checks CI pipeline status
  │
  ▼
AGENT verifies:
  - All tests green
  - Migration is backward-compatible
  - Feature flag is OFF by default
  - Health checks pass post-deploy
  │
  ▼
AGENT reports: "Staging deploy complete. Health: UP. Ready for smoke test."
  │
  ▼
YOU smoke test → approve → AGENT promotes to production
  │
  ▼
AGENT monitors for 15 minutes post-deploy:
  - Error rate stable
  - Latency normal
  - No migration locks
  │
  ▼
AGENT reports: "Production deploy healthy. Rolling out feature flag 10% → 100%."
```

### Example Prompt

> "Deploy v2.4.0 to production. Run smoke tests. Monitor error rate and latency for 15 minutes. Enable feature flag at 10%, monitor 5 min, then 100%. Alert me if anything deviates."

### Deliverables

- [ ] CI pipeline green
- [ ] Database migration applied (no locks)
- [ ] Staging deploy + smoke test passed
- [ ] Production deploy (canary or blue-green)
- [ ] Post-deploy monitoring (15 min observation window)
- [ ] `CHANGELOG.md` updated with release date

---

## Phase 6: Maintenance

**Goal:** Keep it running. Fix bugs. Improve. The cycle never really ends.

### Context Files

| File | Purpose |
|------|---------|
| **RUNBOOK.md** | Ops commands, incident response |
| **MONITORING.md** | Dashboards, metrics, alerts |
| CHANGELOG.md | Release history, deprecation tracking |
| TASKS.md | Active bugs, tech debt, backlog |

### Workflow

```
┌─────────────────────────────────────────┐
│  DAILY                                   │
│  AGENT checks MONITORING.md dashboards   │
│  AGENT reports: "P95 latency normal.     │
│   No errors. 2FA adoption at 34%."      │
├─────────────────────────────────────────┤
│  INCIDENT                                │
│  PagerDuty fires → AGENT reads RUNBOOK   │
│  AGENT triages: "DB connection pool      │
│   exhausted. Kill slow queries?"         │
│  YOU approve → AGENT executes fix        │
│  AGENT writes incident doc              │
├─────────────────────────────────────────┤
│  TECH DEBT                               │
│  AGENT scans TASKS.md weekly             │
│  AGENT proposes: "Top 3 tech debt items  │
│   to tackle this sprint"                │
│  YOU pick → AGENT implements            │
├─────────────────────────────────────────┤
│  DEPRECATION                             │
│  AGENT monitors CHANGELOG.md timelines   │
│  AGENT warns: "v1 token endpoint         │
│   scheduled for removal in 14 days"     │
└─────────────────────────────────────────┘
```

### Example Prompt (Daily Check)
>
> "Check MONITORING.md dashboards. Report anomalies. Check error logs for new patterns. Report 2FA adoption rate. Flag any alerts."

### Example Prompt (Incident)
>
> "PagerDuty: API error rate 12%. Follow RUNBOOK.md incident playbook. Triage. Propose mitigation. Do not execute without approval."

### Deliverables (Ongoing)

- [ ] Daily health check (automated)
- [ ] Incidents documented (in `docs/incidents/`)
- [ ] Tech debt reduced (tracked in `TASKS.md`)
- [ ] Dependency updates applied (security patches)
- [ ] Deprecation timelines honored

---

## Quick Reference: Which File When?

```
=== REQUIREMENTS ===
"I have an idea"              → PRD.md
"Need formal traceability"    → SRS/SRS_TEMPLATE.md

=== DESIGN ===
"How should we build it?"     → DESIGN_SPEC.md (backend) + UI_SPEC.md (frontend)
"Why did we choose X over Y?" → ADR.md
"How do modules interact?"    → HLDD/HLDD_TEMPLATE.md
"How does this component work?" → LLDD/LLDD_TEMPLATE.md
"Show me the architecture"    → DIAGRAMS/ (*.mmd files)
"What colors/fonts do we use?" → UI_SPEC.md
"What routes exist?"          → PAGE_SPEC.md
"What components exist?"      → COMPONENT_TREE.md
"What does the API return?"   → API_CONTRACT.md

=== DEVELOPMENT ===
"Write the code"              → AGENTS.md + CONVENTIONS.md
"Write a database query"      → DATABASE.md
"Add an endpoint"             → API_PATTERNS.md
"What does 'SKU' mean here?"  → GLOSSARY.md

=== TESTING ===
"What's our test strategy?"   → TEST_STRATEGY.md
"Write unit tests"            → UNIT_TEST.md
"Test API + DB together"      → INTEGRATION_TEST.md
"Test full user flow"         → E2E_TEST.md
"Run UAT sign-off"            → UAT.md
"Load test the API"           → PERFORMANCE_TEST.md
"Handle auth / payments"      → SECURITY.md

=== DEPLOYMENT ===
"How do environments work?"   → DEPLOYMENT_WORKFLOW.md
"Set up CI/CD pipeline"       → CI_CD.md
"Deploy to production"        → DEPLOYMENT.md
"How do I run this locally?"  → DEVELOPMENT.md

=== MAINTENANCE ===
"Something is broken"         → RUNBOOK.md
"Is the system healthy?"      → MONITORING.md
"What changed in v2.4?"       → CHANGELOG.md
"What are we working on?"     → TASKS.md
```

---

## The Rules

### For You (Human)

1. **Start with PRD.md.** Always. Code without a PRD is just vibes.
2. **Review everything.** The agent is fast. You're the safety net.
3. **Keep files updated.** Stale context is worse than no context.
4. **One feature per PRD.** Don't cram unrelated things into one doc.

### For The Agent

1. **Read context before writing code.** Never guess.
2. **Ask when unclear.** "I don't know" beats a confident wrong answer.
3. **Follow conventions exactly.** No personal style preferences.
4. **Test before PR.** Never submit untested code.
5. **Flag design mismatches.** If implementation diverges from DESIGN_SPEC.md, say so.

---

## Project Checklist (Copy & Track)

### Bootstrap

- [ ] `AGENTS.md` — stack, commands, conventions
- [ ] `CONVENTIONS.md` — code style, naming, git
- [ ] `.gitignore` — appropriate for stack

### Feature Work (Per Feature)

- [ ] `PRD.md` — what & why
- [ ] `DESIGN_SPEC.md` — how (for non-trivial features)
- [ ] Implementation — per AGENTS.md + CONVENTIONS.md
- [ ] Tests — per TESTING.md
- [ ] `CHANGELOG.md` — unreleased entry
- [ ] PR review → merge

### Production Readiness

- [ ] `DEPLOYMENT.md` — environments & CI/CD
- [ ] `SECURITY.md` — auth rules & constraints
- [ ] `MONITORING.md` — dashboards & alerts
- [ ] `RUNBOOK.md` — ops commands & incident response

---

_Last updated: 2026-06-07_
