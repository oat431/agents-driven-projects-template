# Phase 4: Testing & Quality

**What:** Verify the feature works, doesn't break existing things, and is secure.
**Who:** AI agent generates tests. Human reviews coverage gaps.
**When:** During and after development. Security review before deploy.

## Templates

| File | Use When |
|------|----------|
| `TESTING.md` | Any project. Test patterns, commands, coverage targets. |
| `SECURITY.md` | Auth, payments, PII, or compliance requirements. |

## Workflow

1. Agent reads PRD.md → DESIGN_SPEC.md → TESTING.md → SECURITY.md
2. Agent writes test plan covering user stories + edge cases
3. Agent runs full test suite, reports coverage
4. Agent runs security checklist from SECURITY.md
5. You review gaps → agent fills them
6. CI green → move to Phase 5

## Example Prompt

```
"Review the [feature] implementation against PRD.md.
Identify missing test coverage. Add tests for:
- Happy path (all user stories)
- Edge cases from PRD.md
- Error paths (invalid input, rate limiting, timeouts)
- Security (from SECURITY.md checklist)

Run full test suite. Report coverage vs targets in TESTING.md."
```

## Deliverables

- [ ] Unit tests cover business logic
- [ ] Integration tests cover API contracts
- [ ] Edge cases from PRD.md tested
- [ ] Security checklist from SECURITY.md passed
- [ ] Coverage meets targets
- [ ] Regression suite green
