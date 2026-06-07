# PROJECT_CHECKLIST.md — New Project Bootstrap

<!--
  One-page checklist. Run through this when starting a new project.
  Check off items as you go. Skip what doesn't apply.
-->

## Phase 0: Bootstrap (Day 0)

- [ ] Create project repo
- [ ] Initialize framework scaffold
- [ ] Copy AGENTS.md → fill in stack + commands
- [ ] Copy CONVENTIONS.md → fill in code style + git rules
- [ ] Copy .gitignore → appropriate for stack
- [ ] First commit: `chore: initial bootstrap`

## Phase 1: Requirements (Day 1)

- [ ] Copy PRD.md → fill in problem + user stories
- [ ] Define success metrics (measurable)
- [ ] Document scope boundaries (what's NOT included)
- [ ] Stakeholder review + sign-off
- [ ] (Optional) SRS for formal/compliance projects

## Phase 2: Design (Day 2–3)

- [ ] Copy ARCHITECTURE.md → high-level structure
- [ ] Copy DESIGN_SPEC.md → feature technical design
- [ ] Create diagrams in DIAGRAMS/ (ERD, sequence, deployment)
- [ ] (If UI) Copy UI_SPEC.md → design tokens
- [ ] (If UI) Copy PAGE_SPEC.md → routes + states
- [ ] (If API) Copy API_CONTRACT.md → endpoints
- [ ] (If multi-module) Copy HLDD/ + LLDD/ → decomposition
- [ ] Write first ADR for key decisions
- [ ] Technical review + sign-off

## Phase 3: Development (Day 3–N)

- [ ] Copy DATABASE.md → schema + query conventions
- [ ] Copy API_PATTERNS.md → endpoint conventions
- [ ] Copy GLOSSARY.md → domain terms (if needed)
- [ ] Verify AGENTS.md commands are correct
- [ ] First feature: migration → service → API → test → PR

## Phase 4: Testing (Continuous)

- [ ] Copy TEST_STRATEGY.md → coverage targets
- [ ] Copy UNIT_TEST.md → adjust patterns for your stack
- [ ] Copy INTEGRATION_TEST.md → adjust for your DB
- [ ] Write first test → verify CI runs it
- [ ] (Before UAT) Copy UAT.md → trace to PRD
- [ ] (Before launch) Copy PERFORMANCE_TEST.md → baselines
- [ ] Copy SECURITY.md → run checklist

## Phase 5: Deployment (Before First Release)

- [ ] Copy DEPLOYMENT_WORKFLOW.md → environment map
- [ ] Copy CI_CD.md → pipeline config
- [ ] Copy DEPLOYMENT.md → infrastructure setup
- [ ] Copy DEVELOPMENT.md → local setup docs
- [ ] Test deploy to DEV → smoke test
- [ ] Test rollback procedure
- [ ] Set up monitoring dashboards

## Phase 6: Maintenance (Ongoing)

- [ ] Copy RUNBOOK.md → common ops commands
- [ ] Copy MONITORING.md → dashboards + alerts
- [ ] Copy CHANGELOG.md → start tracking releases
- [ ] Copy TASKS.md → backlog + tech debt
- [ ] Set up daily health check (automated or agent-driven)

---

## Start Small

You don't need all 30+ files on day one. Here's the minimal path:

```
Day 0:  AGENTS.md + CONVENTIONS.md        (2 files)
Day 1:  PRD.md                            (3 files)
Day 3:  DESIGN_SPEC.md + ARCHITECTURE.md  (5 files)
Day 5:  DATABASE.md + UNIT_TEST.md        (7 files)
Week 2: Everything else as needed
```

---

_Last updated: 2026-06-07_
