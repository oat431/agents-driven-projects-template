# API_CONTRACT.md — Frontend-Backend Interface

<!--
  The contract between frontend and backend. AI agents on EITHER side
  use this to stay in sync. The frontend agent knows what to expect.
  The backend agent knows what to deliver.
-->

## Contract Ownership
- **Backend owns** the implementation. The API is the source of truth.
- **Frontend owns** this document. It describes what the frontend expects.
- **Discrepancy** = bug. Fix the backend or update this doc.

## Base Configuration
```
Base URL (dev):  http://localhost:8080/api/v1
Base URL (prod): https://api.panomete.com/v1
Auth:            Bearer <JWT access token>
Content-Type:    application/json
```

## Endpoints

### Auth

#### POST /auth/login
```
Request:
{
  "email": "string",
  "password": "string"
}

Response 200:
{
  "success": true,
  "data": {
    "accessToken": "string (JWT, 15min TTL)",
    "refreshToken": "string (JWT, 24h TTL)",
    "user": {
      "id": "uuid",
      "email": "string",
      "name": "string",
      "role": "USER | ADMIN"
    }
  }
}

Frontend Usage:
- Store accessToken in memory (Pinia). Never localStorage.
- refreshToken set as httpOnly cookie by backend.
- On 401: call /auth/refresh → retry original request.
```

#### POST /auth/refresh
```
Request: (no body — refresh token from cookie)

Response 200:
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string"
  }
}

Frontend Usage:
- Axios interceptor catches 401 → calls refresh → retries.
- If refresh also fails → redirect to /login.
```

### Users

#### GET /users
```
Query: ?page=1&size=20&search=john&role=ADMIN&sort=createdAt,desc

Response 200:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "email": "string",
      "name": "string",
      "role": "USER | ADMIN",
      "isActive": "boolean",
      "createdAt": "ISO 8601"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}

Frontend Usage:
- DataTable with server-side pagination + sorting.
- Search debounced 300ms.
- Role filter: dropdown.
```

### Products

#### GET /products
```
Query: ?page=1&size=20&search=wireless&minPrice=100&maxPrice=500&sort=price,asc

Response 200:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "slug": "string (URL-friendly)",
      "name": "string",
      "description": "string (HTML — sanitized)",
      "price": "number (decimal string from API → parse client-side)",
      "compareAtPrice": "number | null (sale price if set)",
      "images": ["url"],
      "inStock": "boolean",
      "createdAt": "ISO 8601"
    }
  ],
  "meta": { ... }
}
```

## Error Response (All Endpoints)
```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "Human-readable description",
    "details": null
  }
}
```

## Error Codes the Frontend Handles
| Code | Frontend Behavior |
|------|-------------------|
| `VALIDATION_ERROR` | Show field-level errors on form |
| `UNAUTHORIZED` | Redirect to login |
| `FORBIDDEN` | Show "You don't have access" page |
| `NOT_FOUND` | Show 404 state |
| `RATE_LIMITED` | Show toast, disable button for 15s |
| `CONFLICT` | Show "This already exists" message |
| `SERVER_ERROR` | Toast "Something went wrong" + retry |

## Pagination Contract
- **Request:** `page` (1-indexed), `size` (default 20, max 100).
- **Response:** Always includes `meta` with `page`, `pageSize`, `totalItems`, `totalPages`.
- **Frontend:** `DataTable` uses `totalItems` for page count, `page` for current page.

## Real-time / Polling (Future)
- WebSocket for order status updates (planned v2.2).
- Polling (30s interval) for dashboard stats until WebSocket ready.
