# SRS_TEMPLATE.md — Software Requirements Specification

<!--
  IEEE 830-style SRS. Use when PRD isn't formal enough.
  Traceable. Testable. One SRS per module or feature.
  Copy to 1_REQUIREMENT/SRS/SRS_01_FEATURE_NAME.md
-->

## Document Control

- **Document ID:** SRS-XXX
- **Feature:** <!-- e.g., Two-Factor Authentication -->
- **Version:** 1.0
- **Status:** Draft | Review | Approved
- **Traces to:** PRD.md (Section: <!-- e.g., 2FA Feature -->)
- **Author:** <!-- @engineer -->
- **Last Updated:** YYYY-MM-DD

---

## 1. Introduction

### 1.1 Purpose
<!-- One paragraph. What this SRS covers. What system/module it describes. -->

### 1.2 Scope
<!-- What's in this SRS. What related areas are covered elsewhere. -->
- **In scope:** <!-- e.g., TOTP-based 2FA, recovery codes, admin enforcement -->
- **Out of scope:** <!-- e.g., FIDO2/WebAuthn, biometric auth, SMS fallback -->

### 1.3 Definitions

| Term | Definition |
|------|-----------|
| TOTP | Time-based One-Time Password (RFC 6238) |
| 2FA | Two-Factor Authentication |
| Recovery Code | Single-use backup code for account recovery |

---

## 2. Overall Description

### 2.1 User Characteristics
<!-- Who uses this? Technical level? -->
- **Primary users:** All authenticated users. Basic technical literacy assumed.
- **Admin users:** Organization administrators. Higher technical level.

### 2.2 Assumptions & Dependencies

- User has a smartphone with an authenticator app (Google Authenticator, Authy, etc.).
- System clock is synchronized via NTP (required for TOTP).
- Email service is operational (for recovery code distribution alternatives).
- Auth service handles JWT issuance (existing dependency).

---

## 3. Functional Requirements

### FR-01: Enable Two-Factor Authentication

**Priority:** P0 (MVP)  
**Traces to:** US-1 (PRD.md)

**Description:** Authenticated user can enable TOTP-based 2FA on their account.

**Inputs:**

- Valid access token (user must be logged in)
- 6-digit TOTP verification code

**Processing:**

1. System generates a unique TOTP secret (20 bytes, SecureRandom)
2. Secret is encrypted (AES-256-GCM) and stored
3. QR code URI generated (`otpauth://totp/...`)
4. 10 single-use recovery codes generated (16 characters each)
5. Recovery codes hashed (SHA-256) and stored
6. User submits TOTP code for verification
7. On valid code: 2FA marked as active, recovery codes displayed once

**Outputs:**

- Success: `{ qrCodeUri, recoveryCodes[] }`
- Error: `{ code: "TOTP_ALREADY_ENABLED" | "INVALID_CODE" }`

**Acceptance Criteria:**

- [ ] AC-01: QR code displays and can be scanned by standard authenticator apps
- [ ] AC-02: Valid TOTP code activates 2FA
- [ ] AC-03: Invalid TOTP code shows error (max 5 attempts)
- [ ] AC-04: 10 recovery codes generated, displayed exactly once
- [ ] AC-05: Recovery codes are single-use only
- [ ] AC-06: Re-enabling 2FA regenerates secret + recovery codes

---

### FR-02: Login with Two-Factor Authentication

**Priority:** P0 (MVP)  
**Traces to:** US-2 (PRD.md)

**Description:** User with 2FA enabled must provide TOTP code after password.

**Inputs:**

- Email + password (first step)
- 6-digit TOTP code or recovery code (second step)

**Processing:**

1. Validate email + password → issue partial token (5 min TTL, scoped to 2FA)
2. User submits TOTP code or recovery code
3. TOTP: verify against stored secret (allow ±1 time step for clock skew)
4. Recovery code: verify hash, mark as used
5. Issue full access token + refresh token

**Outputs:**

- Success: `{ accessToken, refreshToken, expiresIn }`
- Error: `{ code: "INVALID_CREDENTIALS" | "INVALID_TOTP" | "INVALID_RECOVERY_CODE" }`

**Acceptance Criteria:**

- [ ] AC-07: Correct TOTP code grants access
- [ ] AC-08: Wrong TOTP code denied
- [ ] AC-09: Recovery code grants access + code consumed
- [ ] AC-10: Used recovery code denied on second attempt
- [ ] AC-11: Partial token cannot access non-2FA endpoints
- [ ] AC-12: Partial token expires after 5 minutes

---

## 4. Non-Functional Requirements

### 4.1 Performance

| Metric | Target |
|--------|--------|
| TOTP secret generation | <10ms |
| TOTP code verification | <50ms |
| Recovery code generation (10 codes) | <100ms |
| 2FA login flow (end-to-end) | <500ms |

### 4.2 Security

- TOTP secrets: AES-256-GCM encrypted at rest
- Recovery codes: SHA-256 hashed before storage
- Rate limit: 5 failed 2FA attempts per user per 15 minutes
- Partial token: JWT scoped to `purpose: "2fa_verify"` only
- All communication over HTTPS

### 4.3 Reliability

- 2FA verification: 99.9% uptime
- Recovery code generation: must not fail (blocking operation)
- Clock skew tolerance: ±30 seconds

### 4.4 Usability

- QR code size: minimum 200×200px, scannable at arm's length
- Manual entry key: displayed below QR as fallback
- Recovery codes: copyable, downloadable as text file

---

## 5. External Interface Requirements

### 5.1 API Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | /api/v1/auth/2fa/setup | Generate secret + QR |
| POST | /api/v1/auth/2fa/verify | Verify TOTP, activate 2FA |
| POST | /api/v1/auth/login | Modified: returns partial token if 2FA enabled |
| POST | /api/v1/auth/2fa/authenticate | Verify 2FA code, issue tokens |
| POST | /api/v1/auth/2fa/recovery | Authenticate with recovery code |
| DELETE | /api/v1/auth/2fa | Disable 2FA (requires password) |

### 5.2 Database Changes

See `DESIGN_SPEC.md` for full schema. Summary:

- New table: `user_totp` (encrypted secret, status, timestamps)
- New table: `recovery_codes` (hashed codes, usage tracking)
- New column: `users.two_factor_enabled` (boolean, default false)

---

## 6. Traceability

| Requirement | PRD Ref | Design Ref | Test Ref |
|-------------|---------|------------|----------|
| FR-01 (Enable 2FA) | US-1 | DESIGN_SPEC.md § Data Flow: Enable 2FA | UAT-01, UAT-02, UAT-03 |
| FR-02 (Login with 2FA) | US-2 | DESIGN_SPEC.md § Data Flow: Login with 2FA | UAT-04, UAT-05, UAT-06 |
| FR-03 (Admin enforcement) | US-3 | DESIGN_SPEC.md § Admin Flow | UAT-07, UAT-08 |

---

_Last updated: YYYY-MM-DD_
