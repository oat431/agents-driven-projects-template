# Phase 1: Requirements

**What:** Define what to build before building it.
**Who:** Product + Engineering. AI agent reviews for completeness.
**When:** Before any code is written.

## Templates

| File | Use When |
|------|----------|
| `PRD.md` | Every feature. Problem statement, user stories, success metrics, scope boundaries. |
| `SRS/SRS_TEMPLATE.md` | Formal projects. IEEE 830 spec. Multi-module, compliance, or handoff. |

## Workflow

1. Write PRD.md (or have the agent draft it from your description)
2. Agent reads → asks clarifying questions → identifies edge cases
3. You answer → agent updates PRD.md
4. (Optional) For formal projects: expand PRD into SRS with traceable requirements
5. PRD.md approved → move to Phase 2 (Design)

## Example Prompt

```
"I want to build [describe feature]. Draft a PRD with:
problem statement, 3-5 user stories, success metrics,
edge cases, and explicit scope boundaries (what's NOT included).
Ask me clarifying questions before finalizing."
```

## Deliverables

- [ ] Problem statement clear and data-backed
- [ ] User stories written (As a X, I want Y, so that Z)
- [ ] Success metrics defined (measurable, not vague)
- [ ] Edge cases documented
- [ ] Scope explicit — what's in, what's out
- [ ] Stakeholders aligned
