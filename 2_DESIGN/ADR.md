# ADR.md — Architecture Decision Records

<!--
  Why we made the decisions we made.
  ADRs prevent "why did we do this?" 6 months later.
  Copy this template for each significant decision.
-->

## What is an ADR?

A short document that captures a significant architectural decision, the context, the options considered, and the rationale for choosing one over the others.

---

## ADR Template

### ADR-001: <!-- Title (e.g., Use PostgreSQL over MongoDB) -->

**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXX  
**Date:** YYYY-MM-DD  
**Deciders:** <!-- @person1, @person2 -->

**Context:**
<!-- What problem are we solving? What constraints exist? 2-4 sentences. -->

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Option A | ... | ... |
| Option B | ... | ... |
| Option C | ... | ... |

**Decision:**
<!-- What did we choose? One sentence. -->

**Rationale:**
<!-- Why this option over the others? 2-4 sentences. -->

**Consequences:**
<!-- What becomes easier? What becomes harder? What do we need to do because of this? -->

---

## Example ADRs

### ADR-001: Use PostgreSQL over MySQL

**Status:** Accepted  
**Date:** 2026-06-01  
**Deciders:** @tech-lead, @backend-team

**Context:** Need a relational database for auth and order data. Team has experience with both. System needs JSONB for semi-structured data (user preferences, order metadata).

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| PostgreSQL 16 | JSONB support, stronger SQL compliance, better concurrent writes | Slightly higher memory usage |
| MySQL 8.0 | Team more familiar, lower memory | JSON support weaker, no partial indexes on JSON |
| MongoDB | Flexible schema | No ACID transactions (at the time), no joins |

**Decision:** Use PostgreSQL 16.

**Rationale:** JSONB support eliminates need for a separate document store. Stronger SQL compliance means fewer surprises. Team has enough PostgreSQL experience to be productive.

**Consequences:** Need to learn PostgreSQL-specific tooling (pg_stat_statements, EXPLAIN ANALYZE). JSONB queries require different indexing strategy than regular columns. Positive: can use partial indexes, which MySQL doesn't support.

---

### ADR-002: JWT with RS256 over HS256

**Status:** Accepted  
**Date:** 2026-06-03  
**Deciders:** @security-lead, @backend-team

**Context:** Need to sign authentication tokens. Must support token verification by multiple services (microservice mesh) without sharing a secret key.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| RS256 (asymmetric) | Public key can be shared. Each service verifies independently. Supports JWKS. | Slightly slower to sign. Key management needed. |
| HS256 (symmetric) | Faster. Simpler — one shared secret. | Every service needs the secret. Secret rotation = coordinated deploy. |

**Decision:** Use RS256 with JWKS endpoint at `/.well-known/jwks.json`.

**Rationale:** Microservice architecture means 5+ services need to verify tokens. Sharing a symmetric key across all of them is a security risk and deployment coordination nightmare. RS256 means each service only needs the public key.

**Consequences:** Need a key management process (generate key pair, rotate annually). JWT signing is ~2ms slower per token. JWKS endpoint must be publicly accessible. Positive: can add new verifier services without touching auth service.

---

### ADR-003: TOTP over SMS for Two-Factor Authentication

**Status:** Accepted  
**Date:** 2026-06-07  
**Deciders:** @product-owner, @security-lead

**Context:** Need to add two-factor authentication. Balancing security, user experience, and cost.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| TOTP (authenticator app) | Free (no per-message cost). Works offline. Industry standard (RFC 6238). | User needs to install app. |
| SMS OTP | No app required. Familiar to users. | $0.05/msg. SIM swap risk. Slow delivery. |
| Email OTP | Universal. Free. | Email account = single point of failure for both factors. |
| FIDO2/WebAuthn | Highest security. Phishing-resistant. | Requires hardware key or platform authenticator. V2 feature. |

**Decision:** Use TOTP as primary 2FA method. Revisit SMS as optional fallback in V1.1. Plan FIDO2 for V2.

**Rationale:** TOTP provides strong security at zero per-user cost. No dependency on phone networks. User experience is acceptable — most users already have an authenticator app. SMS evaluated and rejected as primary due to SIM swap risk and per-message cost at scale.

**Consequences:** Need to build QR code setup flow. Need recovery code system (TOTP has no "reset password" equivalent). Documentation must explain how to install authenticator apps. Positive: zero operational cost for 2FA.
