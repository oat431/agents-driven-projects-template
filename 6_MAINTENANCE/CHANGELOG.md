# CHANGELOG.md — Release History & Breaking Changes

  Human-readable release notes. Agents use this to understand version history,
  deprecation timelines, and migration paths. Keep entries dated.

## Versioning

- Format: `MAJOR.MINOR.PATCH` (Semantic Versioning)
- MAJOR: Breaking API changes
- MINOR: New features (backward-compatible)
- PATCH: Bug fixes

---

## [Unreleased]

### Added

- New features going into next release

### Changed

- Modifications to existing features

### Deprecated

- Features that will be removed in next major version

### Fixed

- Bug fixes in progress

---

## [2.4.0] — 2026-06-15

### Added

- Two-factor authentication (TOTP). See [PRD](./PRD.md).
- Rate limiting on login endpoints (5 req / 15 min / IP).
- New endpoint: `POST /api/v1/auth/2fa/setup`.

### Changed

- Login response now includes `partialToken` when 2FA is required.
- Session TTL reduced from 7d to 24h (with refresh).

### Deprecated

- `/api/v1/auth/token` → Use `/api/v1/auth/refresh` instead. Removed in v3.0.

### Fixed

- Race condition on concurrent cart updates (#452).
- Token refresh failing when clock skew >30s (#438).
- N+1 query in `OrderService.getUserOrders()` (#441).

---

## [2.3.1] — 2026-06-01

### Fixed

- Memory leak in WebSocket session tracking (#430).
- Incorrect HTTP status on duplicate email (409 instead of 400).

---

## [2.3.0] — 2026-05-20

### Added

- Webhook support for order status updates.
- Bulk user import endpoint (`POST /api/v1/users/bulk`).

### Changed

- `GET /api/v1/orders` now returns `shippingAddress` by default.
- Database connection pool size increased 20 → 50.

### Security

- Updated Spring Boot 3.3 → 3.4 (CVE-2026-XXXXX).
- Added CSRF protection on state-changing endpoints.

---

## Migration Guide: v2.x → v3.0 (Planned)

### Breaking Changes

1. `/api/v1/auth/token` removed — use `/api/v1/auth/refresh`.
2. `UserResponse.created` renamed to `UserResponse.createdAt`.
3. Pagination `totalCount` renamed to `totalItems`.
4. Minimum Java version: 17 → 21.

### Deprecation Schedule

| Feature | Deprecated | Removed |
|---------|-----------|---------|
| `/api/v1/auth/token` | v2.4.0 | v3.0 |
| `totalCount` field | v2.4.0 | v3.0 |
| XML response format | v2.3.0 | v3.0 |

---

_Template based on [Keep a Changelog](https://keepachangelog.com)._  
_Version links: [Unreleased]: <https://github.com/org/repo/compare/v2.4.0...HEAD>_
