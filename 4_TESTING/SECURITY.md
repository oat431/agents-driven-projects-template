# SECURITY.md — Auth, Threats & Safe Patterns

<!--
  Security rules the agent MUST follow. These are non-negotiable.
  Link from AGENTS.md with high priority.
-->

## Authentication Flow

### JWT (Current)
```
1. POST /api/v1/auth/login  → returns accessToken + refreshToken
2. Access token: 15min expiry, sent as Bearer header
3. Refresh token: 7d expiry, httpOnly cookie, /api/v1/auth/refresh
4. Logout: blacklist access token in Redis until expiry
```

### Token Structure
```json
{
  "sub": "user-uuid",
  "roles": ["USER"],
  "iat": 1680000000,
  "exp": 1680000900,
  "jti": "unique-token-id"
}
```

## Authorization Rules

### Role Hierarchy
```
ADMIN > MANAGER > USER
```

### Endpoint Protection
```java
// Public
@PermitAll
POST /api/v1/auth/login

// Authenticated
@PreAuthorize("isAuthenticated()")
GET /api/v1/users/me

// Role-specific
@PreAuthorize("hasRole('ADMIN')")
DELETE /api/v1/users/{id}

// Ownership check (in service layer, not annotation)
if (!order.getUserId().equals(currentUserId)) throw new ForbiddenException();
```

## Hard Security Rules (Agent Must Follow)

### Never
- ❌ Log credentials, tokens, or PII
- ❌ Return stack traces to clients (return generic error + log internally)
- ❌ Hardcode secrets in any file that touches version control
- ❌ Concatenate user input into SQL (use parameterized queries)
- ❌ Trust client-side validation alone — always validate server-side
- ❌ Generate tokens with predictable seeds or weak randomness

### Always
- ✅ Use `java.security.SecureRandom` (Java) / `crypto.randomBytes()` (Node) for tokens
- ✅ Hash passwords with bcrypt/argon2 — never SHA, never MD5
- ✅ Set `HttpOnly; Secure; SameSite=Strict` on cookies
- ✅ Validate and sanitize all user input at the boundary
- ✅ Use parameterized queries or ORM parameter binding
- ✅ Rate-limit login endpoints (5 attempts / 15 min / IP)

## Sensitive Data Handling

### What Must Be Protected
| Data | Storage | Transit | Logging |
|------|---------|---------|---------|
| Passwords | bcrypt hash | HTTPS only | NEVER |
| Emails | Plaintext (encrypted at rest) | HTTPS | Masked: `t***@e***.com` |
| API keys | Hashed (SHA-256) | HTTPS only | NEVER |
| Payment info | Stripe token only | Stripe.js | NEVER |
| Session tokens | Redis (TTL=15min) | HTTPS + httpOnly cookie | NEVER |

### Data Redaction Examples
```java
// Before logging
log.info("User login: {}", email.replaceAll("(?<=.{2}).(?=.*@)", "*"));
// "jo***@example.com"

// Never
log.info("Token: {}", accessToken); // ❌ VIOLATION
```

## Common Attack Vectors (Agent Must Defend Against)

| Attack | Defense |
|--------|---------|
| SQL Injection | Parameterized queries. No string concatenation. |
| XSS | Output encode. CSP headers. `textContent` not `innerHTML`. |
| CSRF | SameSite cookies. Token-based for state-changing operations. |
| IDOR | Ownership check in service layer. Never trust path variable alone. |
| Mass Assignment | Explicit DTOs. Never bind directly to entities. |
| Timing Attack | Constant-time comparison for tokens (`MessageDigest.isEqual`). |
| Path Traversal | Resolve + verify path is within allowed directory. No `../` escaping. |

## Dependency Security
```bash
# Check for known vulnerabilities (run in CI)
./mvnw dependency-check:check       # OWASP Dependency-Check (Java)
npm audit --production              # Node.js
pip-audit                           # Python
```
