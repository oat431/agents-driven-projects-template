# DEVELOPMENT.md — Local Setup & Workflow

  How to get this project running from zero. Replace all after - with real values.

## Prerequisites

- Java 21 (brew install openjdk@21)
- Node.js 22 (nvm use 22)
- Docker Desktop 4.x
- PostgreSQL 16 (brew install postgresql@16)

## Quick Start

```bash
# 1. Clone & enter
git clone <repo-url>
cd <project-dir>

# 2. Install dependencies
./mvnw dependency:resolve       # Java/Maven
# npm install                    # Node.js
# pip install -r requirements.txt # Python

# 3. Start infrastructure
docker compose up -d             # DB, Redis, etc.

# 4. Run migrations
./mvnw flyway:migrate

# 5. Seed data (optional)
./mvnw spring-boot:run -Dspring-boot.run.arguments="--seed"

# 6. Start dev server
./mvnw spring-boot:run
# App running at: http://localhost:8080
```

## Environment Variables

```bash
# Copy and fill in
cp .env.example .env

# Required:
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname

# Optional (defaults work for dev):
REDIS_URL=redis://localhost:6379
LOG_LEVEL=DEBUG
```

## Testing

```bash
# All tests
./mvnw test

# Specific test
./mvnw test -Dtest=UserServiceTest

# With coverage
./mvnw jacoco:report
# Report at: target/site/jacoco/index.html

# Integration tests (needs Docker running)
./mvnw verify -P integration
```

## Useful Scripts

```bash
# Reset dev database
./scripts/reset-db.sh

# Generate API client types (TypeScript projects)
npm run generate-types

# Lint everything
./mvnw checkstyle:check && npm run lint
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Port 8080 already in use | `lsof -i :8080` → kill the process |
| Flyway migration failed | Check `flyway_schema_history` for conflicts |
| Tests fail locally but pass in CI | `docker compose up -d` — you're missing infra |
