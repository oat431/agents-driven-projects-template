# TESTING.md — (Superseded)

<!--
  ⚠️ This file is superseded by:
  - TEST_STRATEGY.md (master plan: pyramid, coverage, quality gates)
  - UNIT_TEST.md (backend + frontend test patterns)
  - INTEGRATION_TEST.md (API + DB slices)
  - E2E_TEST.md (user flow tests)
  - UAT.md (traced to SRS/PRD)

  Keeping this file for reference. New projects: start with TEST_STRATEGY.md.
-->

## Test Pyramid

```
       ┌──────┐
       │ E2E  │  ~5%    Critical user flows. Cypress / Playwright / Selenium.
       ├──────┤
       │ Int. │ ~20%   API + DB. Spring Boot Test / Supertest / pytest.
       ├──────┤
       │ Unit │ ~75%   Pure logic. JUnit 5 / Vitest / pytest.
       └──────┘
```

## Commands

```bash
# All tests
./mvnw test

# Unit tests only
./mvnw test -Dgroups="unit"

# Integration tests (needs Docker)
./mvnw verify -P integration

# Single test
./mvnw test -Dtest=UserServiceTest

# Coverage
./mvnw jacoco:report
# Report: target/site/jacoco/index.html
```

## Coverage Targets
- Overall: 80% line coverage
- Service layer: 90%+ (business logic)
- Controllers: 70% (thin, tested via integration)
- Models/DTOs: 50% (mostly data, low value)
