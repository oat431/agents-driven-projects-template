# TEST_STRATEGY.md — Master Testing Plan

<!--
  The testing constitution. AI agents read this to understand:
  - What testing looks like in this project
  - Which tests to write for what kind of change
  - Coverage targets and quality gates
-->

## Testing Philosophy

1. **Tests are code.** Same conventions, same review, same quality.
2. **Test behavior, not implementation.** Refactoring shouldn't break tests.
3. **One assertion per test… usually.** If testing a single behavior requires multiple assertions, that's fine. Don't test 5 unrelated things in one test.
4. **Fast feedback.** Unit tests under 1s total. Integration under 30s. E2E under 5 min.
5. **AI writes tests by default.** Every PR with new code includes tests. Non-negotiable.

## Test Pyramid

```
         ┌──────┐
         │ E2E  │  ~5%    Critical user flows. Real browser.
         ├──────┤
         │ UAT  │  ~5%    Traced to SRS user stories.
         ├──────┤
         │ INT. │ ~20%    API + DB + service boundaries.
         ├──────┤
         │ UNIT │ ~70%    Pure logic. No I/O. Fast.
         └──────┘
```

## What Test for What Change?

| Type of Change | Unit | Integration | E2E | UAT |
|---------------|------|-------------|-----|-----|
| New utility function | ✅ | — | — | — |
| New API endpoint | ✅ (controller) | ✅ (full slice) | — | ✅ (if SRS exists) |
| New UI component | ✅ (render + logic) | — | ✅ (user flow) | — |
| Bug fix | ✅ (regression) | ✅ (if boundary fix) | — | — |
| Refactor (no behavior change) | — | — | — | — (existing tests cover) |
| New feature (SRS defined) | ✅ | ✅ | ✅ (if critical) | ✅ |

## Coverage Targets

| Layer | Line Coverage | Branch Coverage | Notes |
|-------|--------------|----------------|-------|
| Service / Business Logic | 90% | 85% | This is where bugs live |
| Controllers / Routes | 75% | 70% | Integration tests cover the rest |
| Models / DTOs / Types | 50% | — | Mostly data — test serialization only |
| UI Components | 80% | 70% | Test rendering + interactions |
| Utilities / Helpers | 95% | 90% | Pure functions — easy to cover |

## Quality Gates (CI Must Pass)

```yaml
# .github/workflows/test.yml
quality-gates:
  - All unit tests pass
  - All integration tests pass
  - Coverage ≥ target (per layer)
  - No skipped tests (@skip)
  - No console.error in test output
  - Lint passes
  - TypeScript compiles (tsc --noEmit)
```

## When Tests Run

| Trigger | Unit | Integration | E2E |
|---------|------|-------------|-----|
| Push to feature branch | ✅ | — | — |
| Open PR | ✅ | ✅ | — |
| Merge to main | ✅ | ✅ | ✅ |
| Nightly (3 AM) | — | — | ✅ (full suite) |
| Before release tag | ✅ | ✅ | ✅ |

## Test Naming Convention

```
Backend:  {ClassName}Test.{methodName}_{stateUnderTest}_{expectedBehavior}
Frontend: {Component}.test.{variant}.{state}_{expectedBehavior}

Examples:
  AuthServiceTest.login_WithValidCredentials_ReturnsTokens
  ProductCard.test.desktop.outOfStock_showsNotifyButton
```

## Rules for AI Agents

1. **Every new file = new test file.** `UserService.java` → `UserServiceTest.java`.
2. **Test edge cases from PRD/SRS.** Not just happy path.
3. **Mock at boundaries.** I/O, external APIs, clock, random. Never mock the class under test.
4. **Tests must be runnable.** `./mvnw test` or `pnpm test` must pass before PR.
5. **Flaky test?** Fix or quarantine. Never merge with `@FlakyTest`.
6. **No test-only dependencies** in production code. Use test fixtures/builders instead.
