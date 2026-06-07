# Phase 2: Design

**What:** Decide how to build before coding. Prevents architecture regret.
**Who:** Engineering lead + team. AI agent drafts, human reviews.
**When:** After PRD approved, before first line of code.

## Templates

| File | Use When |
|------|----------|
| `DESIGN_SPEC.md` | Non-trivial features. Technical design, data flow, API contract, rollout plan. |
| `ARCHITECTURE.md` | Cross-cutting decisions. Design patterns, tech debt, future direction. |

## Which to Use?

- **New feature with API + DB changes?** → Both. DESIGN_SPEC for the feature, ARCHITECTURE for decisions that affect the whole system.
- **Small change, no new endpoints?** → Skip DESIGN_SPEC. Update ARCHITECTURE if a new pattern emerges.
- **Just fixing a bug?** → Neither. Jump to Development.

## Workflow

1. Agent reads PRD.md
2. Agent drafts DESIGN_SPEC.md: component diagram, data flow, API contract, DB schema, rollout plan
3. You review: challenge decisions, evaluate alternatives
4. Agent updates with decisions + rationale
5. Agent updates ARCHITECTURE.md with any new cross-cutting patterns
6. Approved → move to Phase 3

## Example Prompt

```
"Read PRD.md. Design the system architecture. Include:
- Component diagram (which services are involved)
- Data flow for each user story
- API contract (endpoints, request/response)
- Database schema changes
- Security considerations from SECURITY.md
- Rollout plan (feature flags, canary)
Flag any decisions where multiple approaches exist."
```

## Deliverables
- [ ] Component architecture defined
- [ ] API contract specified (if applicable)
- [ ] Database schema designed
- [ ] Security review passed
- [ ] Rollout strategy chosen
- [ ] Architecture decisions documented with rationale
