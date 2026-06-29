# WRN Hub — Product File

**Status:** Internal infrastructure — not a standalone sellable product.

**Last updated:** 2026-06-29
**Source:** WRN Hub v1.1.0 Codebase Capability Audit

---

## Product Role

WRN Hub is internal WebReadyNow infrastructure installed only on webreadynow.com. It is not available for customers to install. It is not marketed or sold as a standalone product.

WRN Hub powers the backend systems that make WebReadyNow plugin licensing, distribution, and documentation work.

---

## What WRN Hub Powers

- WebReadyNow plugin licensing (issue, activate, deactivate, validate)
- Plugin update metadata delivery
- Authenticated plugin download flow
- WooCommerce Subscriptions-to-license automation
- Customer My Account license visibility and download access
- Public documentation library at webreadynow.com/docs
- Documentation search
- Documentation seeding, export, and import between environments

---

## Implemented Capabilities

### Licensing System
- License key issuance tied to WooCommerce orders or subscriptions
- License activation and deactivation by the customer
- License validation endpoint for installed plugins to check against
- License status display in customer My Account area
- License key copy-to-clipboard in My Account

### Plugin Update Infrastructure
- Update metadata endpoint for installed WebReadyNow plugins
- Authenticated download flow — active license required to download latest version from My Account

### WooCommerce Subscriptions Integration
- Subscription status changes can sync with license status when configured
- Renewal, cancellation, and expiry events affect license validity

### Documentation System
- Public documentation pages at webreadynow.com/docs
- Documentation is searchable
- Docs can be seeded programmatically (used during development and environment setup)
- Export/import supported between environments

### Customer-Facing Surfaces
- My Account: license key display, activation status, download links
- Public: webreadynow.com/docs documentation library and search

### Admin-Only Surfaces
- License management dashboard (WebReadyNow internal use only)
- License issuance, manual override, and audit tools
- Docs seeder and import/export tooling

---

## Missing or Unconfirmed Capabilities

- Automatic license key email delivery after purchase — TODO: confirm whether transactional email is triggered post-purchase or requires manual action
- Download protection against direct URL sharing — links are authenticated but full download protection scope is unconfirmed
- Cryptographically secure random key generation — not verified in audit; do not claim
- Customer support ticketing — not part of WRN Hub
- Full customer-facing product portal positioning — WRN Hub has customer surfaces but is not a full self-service portal

---

## Safe Claims

See `03-safe-claims.md` — WRN Hub section for approved claim language.

Summary:
- WRN Hub is the license and update infrastructure for WebReadyNow plugins.
- WebReadyNow issues and manages plugin license keys from webreadynow.com.
- Licenses can be activated, deactivated, and validated through the WebReadyNow license server.
- Customers can view and copy license keys from their My Account area.
- Active license holders can download the latest plugin version from My Account.
- Documentation is publicly available and searchable at webreadynow.com/docs.
- Documentation can be exported and imported between environments.
- Subscription-based license status can sync with WooCommerce Subscriptions when configured.

---

## Unsafe Claims

Do not use:
- WRN Hub is available for customers to install.
- WRN Hub is a standalone product.
- License keys are emailed automatically after purchase.
- Downloads are fully protected against direct URL sharing.
- License keys use cryptographically secure random generation.
- WRN Hub provides customer support ticketing.
- WRN Hub is a full customer-facing product portal.

---

## Product vs. Service Boundary

**WRN Hub handles:** License issuance, activation, validation, update metadata, download authentication, My Account display, public docs, subscription sync when configured.

**WebReadyNow human service handles:** License troubleshooting, manual license overrides, plugin support, subscription/billing edge cases, and any issue that the self-serve My Account flow cannot resolve.

WRN Hub has customer-facing surfaces — My Account licenses and public docs — but the plugin itself is internal admin infrastructure installed only on webreadynow.com.

---

## Recommended Next Agent

**Product & Offer Alignment Agent** — to formally decide whether any WRN Hub capability (e.g., licensing-as-a-service, documentation infrastructure) should become a positioned offer, and to define the relationship between WRN Hub and the plugin products it supports.

Load: `agents/product-offer-alignment-agent.md`
Save output to: `09-agent-outputs/product-alignment/`
