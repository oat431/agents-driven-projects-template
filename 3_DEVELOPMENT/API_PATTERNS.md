# API_PATTERNS.md — Endpoint Conventions & Patterns

  How this project writes APIs. Agents use this to generate consistent endpoints.
  Covers REST, GraphQL, gRPC — whatever you use. Include only what applies.

## REST Conventions

### URL Structure
```
GET    /api/v1/{resource}          — List (paginated)
GET    /api/v1/{resource}/{id}     — Get by ID
POST   /api/v1/{resource}          — Create
PUT    /api/v1/{resource}/{id}     — Full update
PATCH  /api/v1/{resource}/{id}     — Partial update
DELETE /api/v1/{resource}/{id}     — Soft delete
```

### Naming
- Resources: plural nouns (`/users`, `/orders`)
- Nested: max 2 levels (`/orders/{id}/items`)
- Actions: verb in path only for non-CRUD (`/orders/{id}/cancel`, `/invoices/{id}/send`)

### Request/Response Shapes

#### Success Response
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

#### Error Response
```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id 'abc' does not exist",
    "details": null
  }
}
```

#### Pagination
- Query params: `?page=1&size=20&sort=createdAt,desc`
- Default: page=1, size=20
- Max size: 100
- Response includes `meta` with page/size/totalItems/totalPages

### HTTP Status Codes
| Code | When |
|------|------|
| 200 | Success (GET, PUT, PATCH) |
| 201 | Created (POST) |
| 204 | No content (DELETE) |
| 400 | Validation error |
| 401 | Unauthenticated |
| 403 | Forbidden |
| 404 | Resource not found |
| 409 | Conflict (duplicate, state violation) |
| 422 | Unprocessable (valid JSON, invalid semantics) |
| 500 | Unexpected server error |

### Filtering & Search
```
GET /api/v1/users?status=active&role=admin
GET /api/v1/orders?from=2026-01-01&to=2026-06-30
GET /api/v1/products?search=wireless+headphones
```

## Security Patterns

### Authentication
- Header: `Authorization: Bearer <token>`
- Token format: JWT, expires 24h
- Refresh: `POST /api/v1/auth/refresh`

### Authorization
- Role-based: `@PreAuthorize("hasRole('ADMIN')")`
- Resource ownership: checked in service layer
- All endpoints authenticated unless explicitly public

## Breaking Change Policy
- Additive changes: OK anytime (new fields, new endpoints)
- Removing fields: deprecate first (add `deprecated: true`), remove next major version
- Versioning: URL-based (`/api/v1/` → `/api/v2/`). Only bump on breaking changes.
