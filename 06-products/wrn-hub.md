# WRN Hub

**Last updated:** 2026-06-29
**Source:** WRN Hub v1.1.0 Codebase Capability Audit

---

## Role

Internal WebReadyNow infrastructure installed on webreadynow.com.

---

## Not For Sale

WRN Hub is not a customer-installable plugin and should not be marketed as a standalone product.

---

## What It Powers

- License generation and management
- License activation, deactivation, and validation
- Plugin update metadata
- Authenticated plugin download redirects
- WooCommerce Subscriptions to license lifecycle sync
- Customer My Account license visibility
- Public documentation library
- Documentation search
- Documentation seeding, export, and import

---

## Customer-Facing Surfaces

- My Account Licenses tab
- Public documentation at /docs/
- Plugin update notices inside WordPress for licensed products, when client plugins implement update checking

---

## Admin / Internal Surfaces

- Products tab
- Licenses tab
- Docs tab
- WooCommerce product license configuration fields
- Documentation seeder, export, and import tools

---

## Safe Claims

- WebReadyNow uses WRN Hub to manage licenses, updates, downloads, and documentation for its plugins.
- Customers can view and copy their license keys from the WebReadyNow My Account area.
- Active license holders can download their plugin from My Account.
- WebReadyNow plugin documentation is available publicly at webreadynow.com/docs.
- Documentation is searchable.
- Subscription-based license status can sync with WooCommerce Subscriptions when configured.

For the authoritative approved claim list, see `03-safe-claims.md` — WRN Hub section.

---

## Unsafe Claims

- WRN Hub is available for customers to install.
- WRN Hub is a standalone product.
- License keys are emailed automatically after purchase.
- Downloads are fully protected against direct URL sharing.
- License keys use cryptographically secure random generation.
- WRN Hub provides customer support ticketing.

---

## Current Gaps

- License key email delivery is not implemented.
- Admin license search and filtering is not implemented.
- Bulk license operations are not implemented.
- Clearer update-download error messaging is still needed.

---

## Relationship to Other Products

WRN Hub supports WRN Pricing Rules Pro, WRN Store Monitor, and future WebReadyNow plugins by providing the infrastructure needed for licensing, updates, downloads, and documentation.
