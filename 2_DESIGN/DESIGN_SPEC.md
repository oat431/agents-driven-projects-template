# DESIGN_SPEC.md — Technical Design Document

  The HOW. Engineering response to the PRD. Agents use this to understand
  the intended implementation before writing code. Bridge between PRD and AGENTS.md.

## Overview
- **Feature:** [Link to PRD](#)
- **Status:** Draft | Review | Approved | Implementing | Done
- **Author:** @engineer
- **Reviewers:** @tech-lead, @security

## Architecture Decision

### Approach Selected
e.g., "TOTP-based 2FA using java-otp library + encrypted secret storage"

### Alternatives Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| TOTP | Industry standard, no vendor lock-in | User needs authenticator app | ✅ Selected |
| SMS OTP | No app needed | $0.05/msg, SIM swap risk, slow delivery | ❌ Rejected |
| Email OTP | Universal | Email account = single point of failure | ❌ Rejected as primary |

## System Design

### Component Diagram
```
┌──────────┐     ┌──────────────┐     ┌───────────┐
│  Client  │────▶│  API Gateway │────▶│ Auth Svc  │
│ (SPA)    │     │  (validate)  │     │ (verify)  │
└──────────┘     └──────────────┘     └─────┬─────┘
                                            │
                              ┌─────────────┼─────────────┐
                              ▼             ▼             ▼
                        ┌──────────┐ ┌──────────┐ ┌──────────┐
                        │   DB     │ │  Redis   │ │  Email   │
                        │(secrets) │ │(rate lim)│ │(recovery)│
                        └──────────┘ └──────────┘ └──────────┘
```

### Data Flow: Enable 2FA
```
1. User clicks "Enable 2FA" → SPA
2. GET /api/v1/auth/2fa/setup → Auth Svc
3. Generate TOTP secret (SecureRandom, 20 bytes)
4. Store encrypted secret in DB (AES-256-GCM)
5. Generate recovery codes (10 × 16-char random)
6. Hash recovery codes (SHA-256), store in DB
7. Return QR code URI + plaintext recovery codes
8. User scans QR → TOTP app
9. POST /api/v1/auth/2fa/verify (TOTP code)
10. Verify code → Activate 2FA → Issue new JWT with 2fa_verified claim
```

### Data Flow: Login with 2FA
```
1. POST /api/v1/auth/login → email + password
2. Validate credentials → return partial_token (5min TTL, can only complete 2FA)
3. Client redirected to /verify-2fa
4. POST /api/v1/auth/2fa/verify → partial_token + TOTP code
5. Verify TOTP → issue full access_token + refresh_token
```

## Data Model Changes

### New Table: `user_totp`
```sql
CREATE TABLE user_totp (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    encrypted_secret BYTEA NOT NULL,        -- AES-256-GCM encrypted
    encryption_key_id VARCHAR(50) NOT NULL,  -- Which KMS key was used
    is_active BOOLEAN DEFAULT false,
    verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(user_id)  -- One TOTP config per user
);
```

### New Table: `recovery_codes`
```sql
CREATE TABLE recovery_codes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    code_hash VARCHAR(64) NOT NULL,  -- SHA-256 of recovery code
    is_used BOOLEAN DEFAULT false,
    used_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT now()
);
CREATE INDEX idx_recovery_user ON recovery_codes(user_id, is_used);
```

## API Contract

### POST /api/v1/auth/2fa/setup
```
Headers: Authorization: Bearer <access_token>
Response 200:
{
  "success": true,
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qrCodeUri": "otpauth://totp/MyApp:user@example.com?secret=JBSWY3DPEHPK3PXP&issuer=MyApp",
    "recoveryCodes": ["abcd-efgh-ijkl-mnop", ...]
  }
}
Error 409: { "error": { "code": "TOTP_ALREADY_ENABLED" } }
```

### POST /api/v1/auth/2fa/verify
```
Body: { "code": "123456", "partialToken": "eyJ..." }
Response 200:
{
  "success": true,
  "data": {
    "accessToken": "eyJ...",
    "refreshToken": "eyJ...",
    "expiresIn": 900
  }
}
Error 401: { "error": { "code": "INVALID_TOTP_CODE" } }
```

## Security Considerations
- TOTP secrets encrypted at rest (AES-256-GCM, key from KMS)
- Recovery codes: hashed (SHA-256) before storage. Plaintext shown once only.
- Rate limit: 5 failed TOTP attempts per user per 15 minutes
- Partial token: short-lived (5min), scoped to 2FA verify only
- TOTP code replay protection: store last used timestamp, reject same window

## Testing Strategy
- Unit: TOTP generation/verification, recovery code generation
- Integration: Full setup → verify → login → recovery flow
- Security: Rate limiting, token scope validation, encryption test vectors
- Edge: Clock skew ±30s, multiple rapid submissions

## Rollout Plan
1. **Week 1:** Deploy behind feature flag (off for all)
2. **Week 2:** Enable for internal team (dogfooding)
3. **Week 3:** Beta users (opt-in)
4. **Week 4:** Gradual rollout (10% → 50% → 100%)
5. **Week 5:** Remove feature flag, announce GA
