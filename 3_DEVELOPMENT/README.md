# Phase 3: Development

**What:** Write working, tested, reviewable code that follows conventions.
**Who:** AI agent implements. Human reviews and approves.
**When:** After design approved. This is the main loop.

## Templates

| File | Use When | Priority |
|------|----------|----------|
| `AGENTS.md` | Every project. No exceptions. | ★ Must |
| `CONVENTIONS.md` | >1 developer or AI agent writing code | ★ Must |
| `DATABASE.md` | Any project with a database | High |
| `API_PATTERNS.md` | Any project with REST/GraphQL endpoints | High |
| `GLOSSARY.md` | Domain-heavy project | Nice |

## Workflow

1. Agent reads DESIGN_SPEC.md → AGENTS.md → CONVENTIONS.md → DATABASE.md
2. Agent implements: migrations → entities → services → controllers → tests
3. Agent self-reviews: runs tests, lint, checkstyle
4. Agent opens PR with description
5. You review → feedback → agent updates → merge
6. Repeat for next feature

## Example Prompt

```
"Implement US-1 from PRD.md. Read:
- DESIGN_SPEC.md for the technical plan
- AGENTS.md for project conventions
- DATABASE.md for query patterns
- CONVENTIONS.md for code style

Create: migration, entity, repository, service, controller.
Write unit + integration tests. Run test suite.
Open a PR with summary of changes and testing instructions."
```

## Deliverables
- [ ] Feature implemented per design
- [ ] Tests pass (unit + integration)
- [ ] Lint/checkstyle clean
- [ ] PR opened with clear description
- [ ] CHANGELOG.md entry added (Unreleased section)
