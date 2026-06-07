# ARCHITECTURE.md — Design Decisions & Patterns

  The WHY behind the codebase. AI agents use this to understand what not to break.
  Update when you make structural decisions. Link from AGENTS.md.

## Architecture Overview

Diagram or 3-5 sentences describing the high-level structure.

```
[Client] → [API Gateway] → [Service] → [Database]
                         → [Service] → [Cache]
                         → [Service] → [Message Queue]
```

## Key Design Decisions

### Decision 1: Why Spring Boot over Quarkus

**Date:** YYYY-MM-DD
**Context:** What problem were we solving?
**Decision:** What did we choose?
**Tradeoffs:** What did we give up?

### Decision 2: Why PostgreSQL JSONB for flexible schema

**Date:** YYYY-MM-DD
**Context:** What problem were we solving?
**Decision:** What did we choose?
**Tradeoffs:** What did we give up?

### Decision 3: Why Redis for caching

**Date:** YYYY-MM-DD
**Context:** What problem were we solving?
**Decision:** What did we choose?
**Tradeoffs:** What did we give up?

## Patterns & Conventions

### Error Handling

How errors flow through the system.

```
Controller → Service throws BusinessException
           → GlobalExceptionHandler catches
           → Returns ApiResponse with error code + message
```

### Data Flow

How data moves through layers.

```
Request DTO → Controller → Service (domain logic)
                          → Repository (data access)
                          → Entity → Response DTO
```

### Validation

Where validation lives.

- Request validation: `@Valid` on controller DTOs
- Business validation: in Service layer
- DB constraints: last line of defense

## Boundaries & Responsibilities

| Module | Responsibility | Must NOT do |
|--------|---------------|-------------|
| <!-- auth-service --> | Authentication, token management | User profile CRUD |
| <!-- user-service --> | Profile, preferences | Auth decisions |
| <!-- payment-service --> | Transactions, billing | Order fulfillment |

## Tech Debt & Known Issues

Things the agent should be aware of.

- UserService.getById() does N+1 queries — needs JOIN fetch
- No integration tests for payment flow — manual testing only
- application.yml is getting bloated — needs splitting

## Future Direction

Where is this heading? Helps agents make forward-compatible choices.

- Planning to split monolith into microservices Q3 2026
- Moving from REST to gRPC for internal service calls
- Evaluating Redis → Valkey migration
