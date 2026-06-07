# DATABASE.md — Schema, Queries & Data Patterns

  Loaded when agent writes queries, migrations, or touches the data layer.
  Keep under ~3KB. Link from AGENTS.md.

## Connection

- **Engine:** PostgreSQL 16
- **Dev:** `postgresql://postgres:postgres@localhost:5432/myapp_dev`
- **Test:** `postgresql://postgres:postgres@localhost:5432/myapp_test`
- **ORM:** Hibernate 6.x / Prisma 5.x / SQLAlchemy 2.x

## Migration Strategy

- **Tool:** Flyway (Java) / Prisma Migrate (Node) / Alembic (Python)
- **Location:** `src/main/resources/db/migration/`
- **Naming:** `V{version}__{description}.sql` (e.g., `V003__add_user_roles.sql`)
- **Rule:** Never edit existing migrations. Always create new ones.
- **Rollback:** Separate undo script or documented manual steps.

## Schema Conventions

### Naming

- Tables: `snake_case`, plural (`users`, `order_items`)
- Columns: `snake_case`, descriptive (`created_at`, `is_active`)
- PK: `id` — UUID or BIGSERIAL
- FK: `{referenced_table}_id` (`user_id` → `users.id`)
- Indexes: `idx_{table}_{column}` (`idx_users_email`)
- Constraints: `uq_{table}_{column}` (unique), `fk_{table}_{ref}` (foreign key)

### Columns Every Table Has

```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
deleted_at  TIMESTAMPTZ              -- soft delete
```

### Types

- Money: `NUMERIC(19,4)` or `BIGINT` (cents) — never FLOAT
- Enums: `VARCHAR(50)` with CHECK constraint (not PG enum type)
- JSON: `JSONB` for flexible/rarely-queried fields
- Booleans: `BOOLEAN` with `is_` prefix (`is_active`, `is_verified`)
- Dates: `TIMESTAMPTZ` always — never `TIMESTAMP`

## Query Conventions

### Performance Rules

- All foreign keys must have indexes.
- Multi-column indexes: most selective column first.
- No `SELECT *` in production code — list columns explicitly.
- Pagination uses keyset (`WHERE id > :lastId`) not offset for large tables.
- `EXPLAIN ANALYZE` for any query hitting >1000 rows.

### N+1 Prevention

- Java/Spring: `@EntityGraph` or `JOIN FETCH` for eager relationships
- Node/Prisma: `include` with explicit field selection
- Python/SQLAlchemy: `joinedload()` or `selectinload()`

### Soft Delete Pattern

```sql
-- All queries filter deleted:
WHERE deleted_at IS NULL

-- Index includes deleted_at for partial indexes:
CREATE INDEX idx_users_active ON users (email) WHERE deleted_at IS NULL;
```

## Key Tables

Summarize major tables. Agent should understand the data model.

| Table | Purpose | Size | Key Relations |
|-------|---------|------|---------------|
| `users` | Core user identity | ~10K | → profiles, → sessions |
| `profiles` | User display data | ~10K (1:1) | → users |
| `orders` | Purchase records | ~100K | → users, → order_items |
| `order_items` | Line items | ~500K | → orders, → products |

## Common Queries (For Reference)

```sql
-- Get user with active orders
SELECT u.*, o.*
FROM users u
JOIN orders o ON o.user_id = u.id AND o.deleted_at IS NULL
WHERE u.id = :userId AND u.deleted_at IS NULL;

-- Paginated list (keyset)
SELECT * FROM products
WHERE id > :lastId AND deleted_at IS NULL
ORDER BY id ASC
LIMIT 20;
```
