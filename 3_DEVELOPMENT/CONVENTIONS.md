# CONVENTIONS.md — Code Style, Naming & Git

  How this project writes code. Agents use this for every line they generate.
  Be specific — vague conventions are ignored.

## Language-Specific Conventions

### Java

- Java 21. Use records for DTOs, sealed classes for domain types.
- No Lombok. Write constructors/getters or use records.
- Services are `@Service` + interface (for testability).
- Repositories extend `JpaRepository<T, UUID>`.
- Configuration: `application.yml` (not `.properties`).

### TypeScript / Node.js

- TypeScript 5.x, strict mode.
- Prefer `type` over `interface` for data shapes.
- Async/await — never raw promises or callbacks.
- Path aliases: `@/` maps to `src/`.
- Database: Prisma with explicit `select` (never `select: { _all: true }`).

### Python

- Python 3.12+. Type hints on all function signatures.
- Pydantic v2 for data validation/serialization.
- `async def` + `await` for I/O-bound code.
- Black for formatting, Ruff for linting.

## Naming Conventions

### General

- **Clarity over brevity.** `calculateOrderTotalWithTax` beats `calcTot`.
- **No abbreviations** unless universally understood (`id`, `url`, `api`, `db`).
- **Consistent vocabulary.** Don't mix `get`/`fetch`/`retrieve` — pick one.

### By Type

| Thing | Convention | Example |
|-------|-----------|---------|
| Class/Model | PascalCase | `OrderService`, `UserProfile` |
| Method/Function | camelCase | `findById()`, `sendConfirmationEmail()` |
| Variable | camelCase | `orderItems`, `isActive` |
| Constant | UPPER_SNAKE | `MAX_RETRY_ATTEMPTS`, `DEFAULT_PAGE_SIZE` |
| File (Java) | PascalCase | `UserService.java` |
| File (TS/Python) | kebab-case or snake_case | `user-service.ts`, `user_service.py` |
| DB Column | snake_case | `created_at`, `user_id` |
| REST endpoint | kebab-case | `/api/v1/order-items` |
| ENV variable | UPPER_SNAKE | `DATABASE_URL`, `REDIS_HOST` |

## Git Conventions

### Branch Naming

```
feature/JIRA-123-add-payment   — New feature
bugfix/JIRA-456-fix-timeout     — Bug fix
hotfix/security-patch-log4j    — Urgent production fix
chore/update-dependencies      — Maintenance
refactor/extract-auth-module   — No behavior change
```

### Commit Messages

```
type(scope): imperative description

- feat(auth): add refresh token rotation
- fix(orders): prevent double-submit on payment timeout
- refactor(user): extract validation to domain service
- test(payment): add integration test for refund flow
- docs(api): document breaking change policy

Body (optional, wrap at 72 chars):
Why this change. What it fixes. Migration notes if needed.
```

### PR Guidelines

- **Title:** Same format as commits.
- **Description:** What + why + how to test.
- **Size:** Under 400 lines. If bigger, break into stacked PRs.
- **Review:** At least 1 approval. CI must pass.
- **Squash merge** to main — keep history clean.

## Code Review Checklist (For PR Review Agent)

- [ ] Tests pass + new tests for new behavior
- [ ] No hardcoded values that should be config
- [ ] Error handling covers all branches
- [ ] No N+1 queries introduced
- [ ] API changes are backward-compatible (or version bump)
- [ ] Logging at appropriate levels (info for business events, debug for details)
- [ ] No secrets, debug code, or commented-out blocks

## Import/Dependency Order (Java Example)

```java
// 1. JDK
import java.time.Instant;
import java.util.UUID;

// 2. Third-party
import org.springframework.stereotype.Service;
import lombok.RequiredArgsConstructor;

// 3. Internal
import com.example.common.exception.NotFoundException;
import com.example.user.domain.User;
```
