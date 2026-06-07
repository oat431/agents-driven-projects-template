# API_PATTERNS.md — project-api (Spring Boot REST API)

## REST Conventions

### URL Structure
```
GET    /api/v1/users               — List (paginated)
GET    /api/v1/users/{id}          — Get by ID
POST   /api/v1/users               — Create
PUT    /api/v1/users/{id}          — Full update
PATCH  /api/v1/users/{id}          — Partial update
DELETE /api/v1/users/{id}          — Soft delete
POST   /api/v1/auth/login          — Action (non-CRUD)
POST   /api/v1/orders/{id}/cancel  — Sub-resource action
```

### Response Envelope (Every Endpoint)

```json
// Success
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

// Error
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id 'abc-123' does not exist",
    "details": null
  }
}
```

### Controller Template

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<ApiResponse<Page<UserResponse>>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String search,
            @RequestParam(required = false) String role
    ) {
        var result = userService.findAll(page, size, search, role);
        return ResponseEntity.ok(ApiResponse.success(result));
    }

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserResponse>> get(@PathVariable UUID id) {
        return ResponseEntity.ok(ApiResponse.success(userService.findById(id)));
    }

    @PostMapping
    public ResponseEntity<ApiResponse<UserResponse>> create(
            @Valid @RequestBody CreateUserRequest request
    ) {
        var user = userService.create(request);
        return ResponseEntity.status(201).body(ApiResponse.success(user));
    }
}
```

### HTTP Status Codes

| Code | When |
|------|------|
| 200 | Success (GET, PUT, PATCH) |
| 201 | Created (POST) |
| 204 | No content (DELETE) |
| 400 | Validation error (`@Valid` failed) |
| 401 | Missing/invalid token |
| 403 | Valid token, wrong role |
| 404 | Resource not found |
| 409 | Conflict (duplicate email, state violation) |
| 429 | Rate limited |

### Pagination

- Request: `?page=1&size=20&sort=createdAt,desc`
- Default: `page=1, size=20`
- Max: `size=100`
- Response includes `meta` with `page`, `pageSize`, `totalItems`, `totalPages`

### DTO Convention

```java
// Request: records with validation
public record CreateUserRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 2, max = 100) String name,
    @NotBlank @Size(min = 8) String password
) {}

// Response: records, never expose entities
public record UserResponse(
    UUID id,
    String email,
    String name,
    String role,
    boolean isActive,
    Instant createdAt
) {
    public static UserResponse from(User user) {
        return new UserResponse(
            user.getId(), user.getEmail(), user.getName(),
            user.getRole().name(), user.isActive(), user.getCreatedAt()
        );
    }
}
```
