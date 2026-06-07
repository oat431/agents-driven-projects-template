# QUICKSTART.md — 5-Minute AI-SDLC Onboarding

<!--
  Fastest path from zero to AI-assisted project.
  Copy this into your team wiki.
-->

## The 3-File Start (Every Project)

```bash
# 1. Copy the minimum
mkdir docs
cp templates/3_DEVELOPMENT/AGENTS.md docs/
cp templates/3_DEVELOPMENT/CONVENTIONS.md docs/
cp templates/1_REQUIREMENT/PRD.md docs/

# 2. Fill in PRD.md (the WHAT)
#    - Problem statement
#    - 3-5 user stories
#    - Scope boundaries

# 3. Fill in AGENTS.md (the HOW)
#    - Stack, commands, project map
#    - Conventions, constraints
#    - → See examples/ for filled-in versions

# 4. Fill in CONVENTIONS.md (the RULES)
#    - Code style, naming, git conventions
#    - PR checklist

# 5. Start coding. Agent now has context.
```

---

## Phase-by-Phase Cheat Sheet

| I'm at this stage... | I need these files |
|----------------------|-------------------|
| "I have an idea" | `PRD.md` |
| "How should we design this?" | `ARCHITECTURE.md` + `DESIGN_SPEC.md` (+ `UI_SPEC.md` if UI) |
| "Time to code" | `AGENTS.md` + `CONVENTIONS.md` + `DATABASE.md` |
| "Write tests" | `TEST_STRATEGY.md` + `UNIT_TEST.md` |
| "Deploy this" | `DEPLOYMENT_WORKFLOW.md` + `CI_CD.md` |
| "Something broke" | `RUNBOOK.md` + `MONITORING.md` |

---

## First Conversation with Your AI Agent

```
"Read docs/AGENTS.md. Read docs/PRD.md.
I'm building [feature name]. 
Start with the database migration, then the API endpoint, then tests.
Follow the patterns in CONVENTIONS.md and DATABASE.md."
```

---

## Growing the Docs (When to Add More)

| Trigger | Add |
|---------|-----|
| Database has 5+ tables | `DATABASE.md` |
| Building a UI | `UI_SPEC.md` + `PAGE_SPEC.md` |
| Multi-service system | `HLDD/` + `API_CONTRACT.md` |
| First production deploy | `DEPLOYMENT_WORKFLOW.md` + `CI_CD.md` |
| Something broke at 3am | `RUNBOOK.md` + `MONITORING.md` |
| New architect joins | `ADR.md` |
| First UAT cycle | `UAT.md` |

---

## Pro Tips

1. **AGENTS.md under 2KB.** Agents load it into every context window. Be ruthless.
2. **One PRD per feature.** Don't cram. Small, focused, shippable.
3. **Update, don't abandon.** Stale docs are worse than no docs.
4. **Link from AGENTS.md.** That's how agents discover the other files.
5. **Start with examples.** Copy `examples/AGENTS.spring-boot.md`, tweak 10 lines.
