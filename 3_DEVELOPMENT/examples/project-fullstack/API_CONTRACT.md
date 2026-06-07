# API_CONTRACT.md — project-fullstack (E-Commerce Platform)

<!-- Shared truth between packages/api and packages/web -->

## Auth

### POST /api/v1/auth/login
```
Request:  { email, password }
Response: { accessToken, refreshToken, user: { id, email, name, role } }
Errors:   401 INVALID_CREDENTIALS | 429 RATE_LIMITED
```

### POST /api/v1/auth/refresh
```
Request:  (refresh token in httpOnly cookie)
Response: { accessToken, refreshToken }
Errors:   401 TOKEN_EXPIRED → redirect to /login
```

## Products

### GET /api/v1/products
```
Query:    ?page=1&size=20&search=wireless&minPrice=100&maxPrice=500&sort=price,asc
Response: {
  data: [{ id, slug, name, price, compareAtPrice, images[], inStock }],
  meta: { page, pageSize, totalItems, totalPages }
}
Web:      ProductGrid → product cards. Server-side pagination.
```

### GET /api/v1/products/{slug}
```
Response: { id, slug, name, description, price, compareAtPrice, images[], inStock, variants[] }
Web:      Product detail page. SSR (good for SEO). States: loading, not found, error.
```

## Orders

### GET /api/v1/orders
```
Query:    ?page=1&size=20&status=pending&sort=createdAt,desc
Response: {
  data: [{ id, status, total, itemCount, createdAt, user: { id, name } }],
  meta: { ... }
}
Web:      OrderTable with status filter dropdown.
```

### GET /api/v1/orders/{id}
```
Response: {
  id, status, total, items: [{ product, quantity, unitPrice }],
  shippingAddress, timeline: [{ status, timestamp }], createdAt
}
Web:      Order detail page. Timeline component for status history.
```

## Error Codes (Both Sides Agree)

| Code | HTTP | Web Behavior |
|------|------|-------------|
| VALIDATION_ERROR | 400 | Show field errors on form |
| UNAUTHORIZED | 401 | Redirect to /login |
| FORBIDDEN | 403 | Show "No access" page |
| NOT_FOUND | 404 | Show 404 state |
| CONFLICT | 409 | Show "Already exists" |
| RATE_LIMITED | 429 | Toast + disable button 15s |
| SERVER_ERROR | 500 | Toast "Something went wrong" + retry |

## Pagination Contract
- Request: `page` (1-indexed), `size` (default 20, max 100)
- Response: always includes `meta` object
- Frontend DataTable uses `totalItems` for total pages
