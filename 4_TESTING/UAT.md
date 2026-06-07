# UAT.md — User Acceptance Testing (Traced to SRS)

<!--
  User Acceptance Testing. Every test case is traceable to a PRD/SRS user story.
  Run these before release. Product owner signs off.
  AI agents can execute automated UAT and report which stories pass/fail.
-->

## UAT Process

```
1. Read PRD.md / SRS → extract acceptance criteria
2. Write UAT scenarios (this document)
3. Automate what can be automated (E2E tests)
4. Manual testing for the rest (UX feel, visual polish)
5. Product owner sign-off per user story
6. All UAT scenarios pass → release candidate
```

---

## Traceability Matrix

Every UAT scenario links back to a requirement.

| UAT ID | User Story | Scenario | Automated? | Status |
|--------|-----------|----------|------------|--------|
| UAT-01 | US-1: User enables 2FA | Setup via QR code, enters valid TOTP, 2FA activated | ✅ E2E | ⬜ |
| UAT-02 | US-1 | Setup with invalid TOTP code → error shown | ✅ E2E | ⬜ |
| UAT-03 | US-1 | Recovery codes generated (10 codes, single-use) | ✅ E2E | ⬜ |
| UAT-04 | US-2: Login with 2FA | Login → prompt for 2FA → enter code → access granted | ✅ E2E | ⬜ |
| UAT-05 | US-2 | Login → prompt for 2FA → wrong code → denied | ✅ E2E | ⬜ |
| UAT-06 | US-2 | Login → use recovery code → access granted, code consumed | ✅ E2E | ⬜ |
| UAT-07 | US-3: Admin enforces 2FA | Admin toggles "Require 2FA" for org | — | ⬜ |
| UAT-08 | US-3 | User without 2FA is redirected to setup on next login | ✅ E2E | ⬜ |

---

## UAT Scenario Template

### UAT-01: User Enables 2FA via Authenticator App

**Traces to:** US-1 (PRD.md)  
**Priority:** P0 (core feature)  
**Precondition:** User is logged in. 2FA is not yet enabled.

**Steps:**

1. Navigate to Settings → Security → Two-Factor Authentication
2. Click "Enable 2FA"
3. Observe QR code displayed
4. Scan QR code with authenticator app
5. Enter the 6-digit TOTP code shown in the app
6. Click "Verify"

**Expected Result:**

- ✅ "2FA enabled successfully" message appears
- ✅ 10 recovery codes are displayed (one-time view)
- ✅ Status shows "2FA: Active"
- ✅ User can copy/download recovery codes

**Failure Conditions:**

- ❌ Invalid TOTP code: error message "Invalid code. Please try again."
- ❌ Already enabled: button shows "Disable 2FA" instead of "Enable 2FA"
- ❌ Rate limited after 5 failed attempts: "Too many attempts. Please wait 15 minutes."

**Test Data:** User account without 2FA. Valid authenticator app installed.

---

### UAT-02: User Submits Invalid TOTP Code

**Traces to:** US-1 (PRD.md) — edge case  
**Priority:** P1  

**Steps:**

1. Complete UAT-01 steps 1–4
2. Enter an incorrect 6-digit code (e.g., `000000`)
3. Click "Verify"

**Expected Result:**

- ✅ Error: "Invalid code. Please try again."
- ✅ QR code remains visible (not dismissed)
- ✅ Attempt counter increments (shown after 3rd failure)

---

### UAT-04: User Logs In with 2FA

**Traces to:** US-2 (PRD.md)  
**Priority:** P0  

**Precondition:** User has 2FA enabled (from UAT-01).

**Steps:**

1. Log out
2. Navigate to login page
3. Enter email + password → click "Sign In"
4. Observe 2FA prompt (password field replaced by 6-digit input)
5. Open authenticator app, copy current TOTP code
6. Enter code → click "Verify"

**Expected Result:**

- ✅ Redirected to dashboard
- ✅ JWT contains `2fa_verified` claim
- ✅ Session persists (no re-prompt for 2FA on page refresh)

---

## UAT Checklist by Feature

### Feature: Two-Factor Authentication

- [ ] UAT-01: Enable 2FA (happy path)
- [ ] UAT-02: Invalid TOTP code
- [ ] UAT-03: Recovery code generation + single-use validation
- [ ] UAT-04: Login with 2FA (happy path)
- [ ] UAT-05: Login with wrong 2FA code
- [ ] UAT-06: Login with recovery code
- [ ] UAT-07: Admin enforces 2FA
- [ ] UAT-08: Non-2FA user redirected to setup
- [ ] UAT-09: "Trust this device for 30 days" (V1.1)
- [ ] UAT-10: Disable 2FA (requires current password)

---

## Rules for AI Agents

1. **Every SRS user story → at least 1 UAT scenario.** Traceable. Verifiable.
2. **Automate the automatable.** E2E tests cover happy path + critical error paths.
3. **Manual for UX.** Visual polish, animation feel, accessibility — human eye required.
4. **Failed UAT = no release.** Fix → re-test → pass → release.
5. **UAT doc is living.** Add scenarios as edge cases are discovered.
