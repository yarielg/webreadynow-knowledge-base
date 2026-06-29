# Codebase Capability Audit: WRN Hub v1.1.0

## Date
2026-06-29

## Agent Used
Codebase Capability Auditor Agent

## Audit Scope
WRN Hub v1.1.0 codebase — includes licensing system, plugin update infrastructure, WooCommerce Subscriptions integration, documentation system, and customer My Account surfaces. Admin-only tooling reviewed.

---

## Executive Capability Decision

WRN Hub is internal WebReadyNow infrastructure. It is not a standalone sellable product. It is safe to describe as the licensing and update infrastructure for WebReadyNow plugins, the system that manages customer My Account license display and downloads, and the platform hosting webreadynow.com/docs.

No marketing copy, product pages, or sales claims should position WRN Hub as a customer-installable product. The customer-facing surfaces (My Account, public docs) are safe to reference in service and product context.

---

## Implemented Capabilities

| Capability | Classification | Safe Claim |
|---|---|---|
| License key issuance | Implemented | WebReadyNow issues plugin license keys from webreadynow.com |
| License activation | Implemented | Licenses can be activated through the WebReadyNow license server |
| License deactivation | Implemented | Licenses can be deactivated through the WebReadyNow license server |
| License validation endpoint | Implemented | Installed plugins validate against the WebReadyNow license server |
| My Account license display | Implemented | Customers can view and copy license keys from My Account |
| My Account download access | Implemented | Active license holders can download the latest plugin version from My Account |
| Plugin update metadata endpoint | Implemented | WebReadyNow plugins receive update metadata from webreadynow.com |
| Authenticated download flow | Implemented | Plugin downloads require an active license |
| Public documentation library | Implemented | Documentation is publicly available at webreadynow.com/docs |
| Documentation search | Implemented | Documentation is searchable |
| Docs seeding | Implemented | Documentation can be seeded programmatically |
| Docs export/import | Implemented | Documentation can be exported and imported between environments |
| WooCommerce Subscriptions sync | Requires configuration | Subscription status can sync with license status when configured |

---

## Partially Implemented Capabilities

**Subscription-to-license automation**
- What exists: Hooks into WooCommerce Subscriptions status events (renewal, cancel, expiry)
- What is missing: Full edge-case coverage unconfirmed; manual override may be needed in some billing scenarios
- Safe wording: "Subscription-based license status can sync with WooCommerce Subscriptions when configured"
- Unsafe wording: "Automatically manages all subscription licensing scenarios"

---

## UI-Only / Stub / Demo Capabilities

None identified in this audit.

---

## Missing Capabilities

- **Automatic post-purchase license email** — TODO: confirm whether a transactional email is triggered after purchase or requires manual/third-party trigger
- **Customer support ticketing** — not present; not in scope for WRN Hub
- **Full self-service portal** — WRN Hub has customer surfaces but is not a complete self-service portal

---

## Requires Configuration

- WooCommerce Subscriptions sync requires WooCommerce Subscriptions to be active and configured
- Authenticated downloads require an active license in the system

---

## Requires Human Service

- License troubleshooting and manual override
- Billing and subscription edge cases
- Plugin support issues beyond My Account self-serve

---

## Claim Safety Review

| Claim | Classification |
|---|---|
| License server for WebReadyNow plugins | Ready to claim |
| My Account license visibility and download | Ready to claim |
| Public docs at webreadynow.com/docs | Ready to claim |
| Documentation is searchable | Ready to claim |
| Subscription license sync | Safe with wording limits — "when configured" required |
| Automatic post-purchase email | Unsafe — unconfirmed; TODO |
| Download protection against URL sharing | Unsafe — scope unconfirmed |
| Cryptographically secure key generation | Unsafe — not verified |
| Customer support ticketing | Unsafe — not implemented |
| Standalone customer-installable product | Unsafe — internal infrastructure only |

---

## Product vs. Service Boundary

**WRN Hub handles:** License issuance, activation, deactivation, validation, update metadata, authenticated downloads, My Account display, public docs, subscription sync when configured.

**WebReadyNow human service handles:** License troubleshooting, manual overrides, plugin support, billing edge cases, and any issue the self-serve flow cannot resolve.

WRN Hub has customer-facing surfaces, but the plugin itself is internal admin infrastructure installed only on webreadynow.com.

---

## Development Roadmap Recommendations

**Must confirm before claiming:**
- Post-purchase license email trigger (confirm or build)

**Should clarify soon:**
- Download URL protection scope and limits

**Nice to have:**
- Subscription edge-case coverage documentation

**Not needed now:**
- Customer support ticketing (out of scope for WRN Hub)

---

## Recommended Next Agent

**Product & Offer Alignment Agent** — to formally decide the positioning of WRN Hub relative to the plugin products it supports, and whether any infrastructure capability (e.g., licensing infrastructure) should become a positioned service offer.

Load: `agents/product-offer-alignment-agent.md`
Save output to: `09-agent-outputs/product-alignment/`

---

## Knowledge Base Update Recommendation

- **Update level:** Level 2
- **Files updated:** `06-products/wrn-hub.md`, `03-safe-claims.md`
- **Decision record:** Not required at this level; audit record serves as the decision artifact
- **Output saved to:** `09-agent-outputs/codebase-audits/wrn-hub-v1.1.0-audit.md`
