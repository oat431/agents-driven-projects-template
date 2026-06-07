# GLOSSARY.md — Domain Language & Business Terms

<!--
  Domain-specific vocabulary. Agents use this to understand what things mean
  in your business context and to communicate with the right terminology.
-->

## Core Concepts

| Term | Definition | Code Name |
|------|-----------|-----------|
| <!-- User --> | <!-- Person with an account. Must verify email. --> | `User` / `users` |
| <!-- Customer --> | <!-- User who has made at least 1 purchase --> | `Customer` (extends User) |
| <!-- Order --> | <!-- Purchase request. States: PENDING → CONFIRMED → SHIPPED → DELIVERED --> | `Order` / `orders` |
| <!-- Shipment --> | <!-- Physical package. One order can have multiple shipments. --> | `Shipment` / `shipments` |
| <!-- Merchant --> | <!-- Seller on the platform. Has store + products. --> | `Merchant` / `merchants` |

## State Machines

### <!-- Order States -->
```
PENDING → CONFIRMED → SHIPPED → DELIVERED
    ↓         ↓
CANCELLED   REFUNDED
```
- **PENDING:** Payment not yet confirmed.
- **CONFIRMED:** Paid, awaiting fulfillment.
- **SHIPPED:** Handed to carrier. Tracking number exists.
- **DELIVERED:** Carrier confirmed drop-off.
- **CANCELLED:** Before CONFIRMED. Full refund.
- **REFUNDED:** After CONFIRMED. Money returned.

### <!-- User States -->
```
INACTIVE → ACTIVE → SUSPENDED
                ↓
            DELETED (soft)
```

## Abbreviations (Internal)
| Abbrev | Meaning |
|--------|---------|
| <!-- SKU --> | Stock Keeping Unit — unique product variant identifier |
| <!-- SLA --> | Service Level Agreement — 99.9% uptime for API |
| <!-- MTTR --> | Mean Time To Recovery — target <15 min for P0 incidents |
| <!-- TTL --> | Time To Live — used for cache expiry and token expiry |

## Departments / Teams
| Team | Owns | Slack Channel |
|------|------|---------------|
| <!-- Platform --> | API, auth, infrastructure | `#team-platform` |
| <!-- Payments --> | Billing, invoices, refunds | `#team-payments` |
| <!-- Core Product --> | User-facing features | `#team-product` |

## External Services (Vendor Names)
| Service | Purpose | Docs |
|---------|---------|------|
| <!-- Stripe --> | Payment processing | https://docs.stripe.com |
| <!-- SendGrid --> | Transactional email | https://docs.sendgrid.com |
| <!-- Auth0 --> | SSO for enterprise customers | https://auth0.com/docs |
| <!-- Cloudflare --> | DNS, CDN, WAF | https://developers.cloudflare.com |
