# AGENTS.md — Project Context for AI Coding Agents

  Drop this in your project root. It's read by OpenClaw, Claude Code, Codex, Aider, 
  Cursor, and others. Keep it under ~2KB — agents load this into context every turn.

## Project
- **Name:** e.g., auth-service
- **Purpose:** One sentence — what problem does this solve?

## Stack
- **Language:** e.g., Java 21, TypeScript 5.x, Python 3.12
- **Framework:** e.g., Spring Boot 3.3, Next.js 14, FastAPI
- **Database:** e.g., PostgreSQL 16, MongoDB 7
- **Runtime:** e.g., JVM, Node.js 22, Docker Compose

## Project Map
Point the agent at what matters. No need to list every file.
```
src/main/java/com/example/   — Application entrypoint & config
src/main/java/com/example/api/ — REST controllers
src/main/java/com/example/service/ — Business logic
src/main/java/com/example/repository/ — Data access
src/main/resources/          — Config files, migrations
tests/                       — Test suite
docs/                        — Architecture & API docs
```

## Commands
Exact commands the agent should run. Include flags that matter.

```bash
# Build
./mvnw clean package -DskipTests

# Test (all)
./mvnw test

# Test (single class)
./mvnw test -Dtest=UserServiceTest

# Run locally
./mvnw spring-boot:run

# Lint
./mvnw checkstyle:check
```

## Conventions
Rules the agent MUST follow. Be specific.
- Java 21. Use records for DTOs. No Lombok.
- Controllers return `ResponseEntity<ApiResponse<T>>`.
- Service methods throw `BusinessException` — never return null.
- Tests use JUnit 5 + AssertJ. Test class naming: `*Test.java`.
- Database migrations in `src/main/resources/db/migration/` (Flyway).
- Commit messages: `type(scope): description` (conventional commits).

## Constraints
Hard boundaries. Agent should not cross these.
- Do NOT modify `application-prod.yml`.
- Do NOT change database migration files — only add new ones.
- Do NOT upgrade framework major version without asking.
- API responses must stay backward-compatible.

## External Dependencies
Services/APIs this project talks to.
- e.g., payment-gateway (internal, port 8082)
- e.g., SendGrid API (email)
- e.g., Redis (localhost:6379) for caching
