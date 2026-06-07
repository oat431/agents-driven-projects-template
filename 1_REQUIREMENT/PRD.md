# PRD.md — Product Requirements Document

  The WHAT and WHY. Before any code exists. Agents use this to validate
  that implementation matches intent. Link from AGENTS.md with top priority.

## Overview (Example change to your project)
- **Feature:** Two-Factor Authentication
- **Status:** Draft | Review | Approved | In Development | Shipped
- **Stakeholders:** @pm-lead, @tech-lead, @design-lead
- **Target Release:** Q3 2026 / v2.4.0

## Problem Statement
What user pain are we solving? Be specific with data if possible.
e.g., "23% of support tickets are password resets. Current reset flow takes 4 emails."

## Success Metrics
How we know this worked. Must be measurable.
- [ ] Password reset support tickets drop by 50% within 30 days
- [ ] 2FA adoption rate > 60% within 90 days
- [ ] Account takeover incidents drop to near-zero

## User Stories

### Primary Flow
```
As a <user type>
I want <goal>
So that <value>
```

- [ ] **US-1:** As a user, I want to enable 2FA via authenticator app, so my account is more secure.
- [ ] **US-2:** As a user, I want backup recovery codes, so I can access my account if I lose my phone.
- [ ] **US-3:** As an admin, I want to require 2FA for all staff accounts, so we meet compliance.

### Edge Cases
- What happens when user loses phone + recovery codes?
- What happens during timezone/DST changes for TOTP?
- How does login via SSO interact with 2FA?
- What happens to existing sessions when 2FA is enabled?

## Functional Requirements

### Must Have (MVP)
- [ ] FR-1: Users can enable/disable TOTP-based 2FA
- [ ] FR-2: Login flow requires 2FA code after password
- [ ] FR-3: Recovery codes generated on 2FA setup (10 single-use codes)
- [ ] FR-4: Admins can enforce 2FA per organization

### Should Have (V1.1)
- [ ] FR-5: SMS fallback option
- [ ] FR-6: Trust this device for 30 days
- [ ] FR-7: 2FA setup wizard with QR code

### Won't Have (Explicitly Out of Scope)
- ❌ Hardware security keys (FIDO2/WebAuthn) — V2
- ❌ Biometric auth — V2
- ❌ Backup phone number for recovery — evaluated, rejected (SIM swap risk)

## Non-Functional Requirements
- **Performance:** 2FA verification under 200ms p95
- **Reliability:** Recovery code generation must not fail (blocking operation)
- **Security:** TOTP secrets encrypted at rest (AES-256-GCM)
- **Accessibility:** QR code setup must have manual key entry fallback
- **Compliance:** SOC 2 requirement — all admin accounts must have 2FA

## Dependencies
- Need design team for 2FA setup UI (by June 15) -->
- Need security team review of TOTP implementation (by June 20) -->
- Depends on auth-service refactor (PR #342) -->

## Open Questions
- [ ] Should we force 2FA on next login or give a grace period?
- [ ] Recovery codes: show once or allow re-display behind re-auth?
- [ ] Do we need to support hardware tokens in MVP for enterprise customers?

## Decisions Log
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-06-01 | TOTP over SMS as primary | Lower cost, no phone dependency, better security |
| 2026-06-03 | 10 recovery codes, single-use | Industry standard. Regenerating resets all. |
