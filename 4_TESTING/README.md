# Phase 4: Testing & Quality

**What:** Verify the feature works, doesn't break existing things, and meets requirements.  
**Who:** AI agent generates tests. Human reviews coverage gaps and UAT sign-off.  
**When:** During and after development. Full suite before release.

---

## Template Index

```
4_TESTING/
│
├── 📄 TEST_STRATEGY.md          ← Master plan: pyramid, coverage targets, quality gates
│
├── 🔬 TEST TYPES
├── 📄 UNIT_TEST.md              ← Backend (JUnit/pytest) + Frontend (Vitest/vue-test-utils)
├── 📄 INTEGRATION_TEST.md       ← API + DB slices. Full stack, real database.
├── 📄 E2E_TEST.md               ← User flows in real browser (Playwright/Cypress)
├── 📄 UAT.md                    ← Traced to SRS/PRD user stories. Sign-off checklist.
├── 📄 PERFORMANCE_TEST.md       ← Load, stress, benchmarks (k6/Artillery)
│
└── 🔒 SECURITY.md               ← Security review checklist (existing ✅)
```

---

## Which Test When?

| I'm building... | Write This |
|-----------------|------------|
| New utility function | UNIT_TEST (pure logic) |
| New API endpoint | UNIT_TEST (controller) + INTEGRATION_TEST (full slice) |
| New UI component | UNIT_TEST (render + interaction) |
| New user flow | E2E_TEST (happy path) |
| New feature (SRS exists) | UNIT + INTEGRATION + UAT (traceable) |
| Performance-sensitive code | PERFORMANCE_TEST (baseline + thresholds) |
| Auth/payment/PII feature | SECURITY.md checklist |

---

## How They Connect

```
PRD.md / SRS           TEST_STRATEGY.md
     │                      │
     │   user stories       │   coverage targets
     ▼                      ▼
   UAT.md    ◀───────  UNIT_TEST.md
 (sign-off)           INTEGRATION_TEST.md
     │                E2E_TEST.md
     │                      │
     │   UAT-01:            │   test cases
     │   verify 2FA    ─────┘   verify code
     │   setup                  matches spec
     ▼
  Release Candidate
```

---

## Workflow

### For a New Feature (Full Cycle)

1. Agent reads PRD.md / SRS → extracts user stories
2. Agent writes UNIT_TEST for business logic + UI components
3. Agent writes INTEGRATION_TEST for API boundaries
4. Agent writes E2E_TEST for critical user flow
5. Agent writes UAT scenarios traced to SRS user stories
6. Agent runs full suite → fixes failures → reports coverage
7. Human reviews UAT → signs off → feature complete

### Example Prompt (Full Testing)

```
"Write a complete test suite for the 2FA feature:

1. Read PRD.md user stories US-1, US-2, US-3
2. UNIT_TEST.md: test TotpService, JwtService changes
3. INTEGRATION_TEST.md: test login → 2FA prompt → verify → dashboard
4. E2E_TEST.md: full Playwright flow for enable + login + recovery
5. UAT.md: traceable scenarios for each user story
6. Run all tests. Fix failures. Report coverage vs TEST_STRATEGY.md targets."
```

### For a Bug Fix

```
"This bug: [describe]. Write a regression test that fails before the fix
and passes after. Follow UNIT_TEST.md patterns. Run full suite to ensure
nothing else broke."
```

---

## Test Data Policy

| Environment | Data Source | Cleanup |
|-------------|------------|---------|
| Unit test | Inline fixtures or builders | Automatic (per test) |
| Integration test | TestContainers / seeded test DB | Before each test |
| E2E test | Dedicated test users + seed scripts | After test suite |
| UAT | Staging environment | Manual reset weekly |

---

## Deliverables

- [ ] TEST_STRATEGY.md defines coverage targets
- [ ] Unit tests cover business logic (≥80%)
- [ ] Integration tests cover API boundaries
- [ ] E2E tests cover critical user flows
- [ ] UAT scenarios traceable to SRS/PRD
- [ ] Performance baselines established
- [ ] Security checklist passed
- [ ] All CI quality gates green
