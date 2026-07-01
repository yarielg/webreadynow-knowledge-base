# WebReadyNow — Offer Map

**Purpose:** Read this file when making decisions about product direction, service packaging, CTAs, pricing models, or business development. It maps every current offer and shows how products and services connect.

---

## Offer Model Matrix

| Offer | Type | Delivery | Current Status |
|---|---|---|---|
| WRN Store Monitor (Free) | Product | WordPress.org, self-serve | TODO — not yet built as standalone plugin |
| WRN Store Monitor Pro — Annual | Product | webreadynow.com, self-serve, BYOK AI | Pre-release — first sellable SKU |
| WRN Store Monitor Pro — Monthly | Product | webreadynow.com, self-serve, WRN-managed AI credits | Blocked — needs WRN Proxy API / AI metering |
| WRN Store Monitor Pro — Managed | Recurring service | Pro plugin + WebReadyNow monthly review | Blocked — needs WRN Proxy API / AI metering + defined SLA |
| WRN Pricing Rules Pro — Plugin only | Product | Self-serve | Active |
| WRN Pricing Rules — Plugin + Setup | Productized service | Plugin + configuration | TODO — define scope |
| WRN Pricing Rules — B2B Implementation | Service | Custom | Available |
| WooCommerce Store Audit | Productized service | Fixed deliverable | Active |
| Performance and Checkout Rescue | Service | Custom scope | Active |
| WooCommerce Integrations and System Sync | Service | Custom scope | Active |
| E-commerce Automation and Operations | Service | Custom scope | Active |
| Custom WooCommerce Plugin Development | Service | Custom scope | Active |
| Miami WooCommerce Developer | Local SEO / service | Custom scope | Active |
| Managed WooCommerce Monitoring | Recurring service | Monthly | Active (with WRN Store Monitor) |

---

## Products

### WRN Store Monitor (Free) + WRN Store Monitor Pro

**What it is:** Two standalone plugins, not one plugin with a paywall.
- **WRN Store Monitor (Free)** — WordPress.org. Basic WooCommerce detection and email alerting. No license key, no AI. Acquisition layer.
- **WRN Store Monitor Pro** — webreadynow.com, license-gated via WRN Hub. Advanced monitoring, alert reliability, AI-assisted diagnosis, evidence tooling. Standalone — does not require Free to be installed. The current advanced codebase (v1.17.3) becomes Pro.

**Current Pro version:** 1.17.3 (Free plugin not yet built — see `06-products/wrn-store-monitor.md`)

**Offer models:**
1. Free plugin-only — self-serve, WordPress.org
2. Pro — Annual (BYOK) — self-serve, first sellable SKU, no WRN AI infrastructure required
3. Pro — Monthly (WRN-managed AI credits) — blocked until WRN Proxy API / AI metering exists
4. Pro — Managed (WRN-managed AI credits + Managed WooCommerce Monitoring) — blocked until WRN Proxy API / AI metering AND a defined Managed Monitoring SLA both exist

**Who buys it:**
- WooCommerce store owners who want early warning on operational issues (Free entry point)
- Agencies managing WooCommerce stores for clients (Pro)
- Stores with meaningful revenue at risk from checkout or payment failures (Pro, especially Managed)

**Key value:**
- Free: detect failed/stuck orders, payment failure patterns, inventory issues, environment/HPOS/email-deliverability checks, health score, email alerts
- Pro: adds continuous checkout/cart/AJAX probing, Slack alerting, alert reliability logging, incident timeline, diagnostic export, AI-assisted triage and diagnosis (BYOK or WRN-managed depending on plan), Code Lab

**Primary CTA:** Install WRN Store Monitor (Free) / Get WRN Store Monitor Pro — Annual / Get Managed Monitoring

**Connects to service:** Managed WooCommerce Monitoring — wraps the Pro — Managed plan when issues require diagnosis, root-cause repair, or ongoing expert support

Full details, feature split table, and commercial structure decision in `06-products/wrn-store-monitor.md` and `09-agent-outputs/product-alignment/wrn-store-monitor-free-pro-commercial-structure.md`.

---

### WRN Pricing Rules Pro / Price List Manager

**What it is:** WooCommerce B2B and wholesale pricing plugin.

**Offer models:**
1. Plugin-only
2. Plugin + Setup
3. B2B Pricing Implementation (service)
4. Custom pricing workflow development

**Who buys it:**
- B2B WooCommerce stores
- Wholesalers and distributors
- Stores with VIP or customer-specific pricing
- Manufacturers selling direct to different buyer types

**Key value:**
- Wholesale and role-based pricing
- Customer-specific price lists
- B2B/B2C pricing separation
- VIP and distributor pricing

**Primary CTA:** Get WRN Pricing Rules / Plan My B2B Pricing

**Connects to service:** B2B Pricing Implementation, Custom WooCommerce Plugin Development

Full details in `06-products/wrn-pricing-rules.md`.

---

### WRN Hub

TODO: Define product purpose, current capability, intended buyers, and relationship to WebReadyNow services.

Full details (when available) in `06-products/wrn-hub.md`.

---

### WebReadyNow Theme

TODO: Define theme purpose, intended use, relationship to WebReadyNow site builds, and whether it is internal-only or sellable.

Full details (when available) in `06-products/webreadynow-theme.md`.

---

## Services

### WooCommerce Store Audit
**Pain it solves:** Store owners spend money rebuilding, adding plugins, or guessing at fixes without knowing the real problem.
**What we deliver:** Diagnosis of WooCommerce issues before spending money guessing.
**Primary CTA:** Request a Store Audit
**Connects to:** All other services — the audit often reveals the next engagement

### Performance and Checkout Rescue
**Pain it solves:** Slow stores, checkout failures, payment issues, caching problems.
**What we deliver:** Investigation and fix of specific WooCommerce performance and checkout problems.
**Primary CTA:** Get Checkout Help / Request a Store Audit
**Connects to:** Store Audit (diagnosis), Managed Monitoring (post-rescue)

### WooCommerce Integrations and System Sync
**Pain it solves:** WooCommerce disconnected from ERP, inventory, fulfillment, or financial systems.
**What we deliver:** Integration between WooCommerce and external systems (ShipStation, QuickBooks, SAP Business One, ERP, fulfillment).
**Primary CTA:** Plan My Integration
**Connects to:** Automation and Operations, Custom Plugin Development

### E-commerce Automation and Operations
**Pain it solves:** Manual, error-prone WooCommerce operations.
**What we deliver:** Automation of WooCommerce operations without creating fragile workflows.
**Primary CTA:** Schedule a WooCommerce Review
**Connects to:** Integrations, Custom Plugin Development

### Custom WooCommerce Plugin Development
**Pain it solves:** Off-the-shelf plugins create risk, complexity, or don't fit the store's workflow.
**What we deliver:** Custom WooCommerce functionality built specifically for the client's store.
**Primary CTA:** Build a Custom WooCommerce Feature
**Connects to:** All services — custom plugins often emerge from audits, rescue, or integration work

### Miami WooCommerce Developer
**Pain it solves:** Local businesses want a WooCommerce expert they can reach and trust.
**What we deliver:** WooCommerce development, optimization, and support for Miami businesses.
**Primary CTA:** Schedule a WooCommerce Review
**Connects to:** Store Audit, all services

### Managed WooCommerce Monitoring
**Pain it solves:** Store owners can't watch their WooCommerce store 24/7.
**What we deliver:** WRN Store Monitor plus WebReadyNow human expertise — monthly alert review, issue diagnosis, and store reliability support.
**Primary CTA:** Get Managed Monitoring
**Connects to:** WRN Store Monitor (plugin), Store Audit, Checkout Rescue

---

## How Products and Services Connect

```
Store owner discovers WRN Store Monitor (Free) on WordPress.org
    → Installs Free plugin — basic detection and email alerts
    → Sees an alert they can't diagnose, or wants advanced monitoring/AI
    → Upgrades to WRN Store Monitor Pro — Annual (BYOK, self-serve, webreadynow.com)
        → Later: Pro — Monthly or Pro — Managed once WRN Proxy API / AI metering exists
    → WebReadyNow offers: Store Audit or Managed Monitoring
        → Audit reveals deeper issues
        → Rescue or Integration engagement
        → Custom plugin if needed
        → Managed Monitoring retainer (Pro — Managed plan)
```

```
B2B store needs wholesale pricing
    → Buys WRN Pricing Rules Pro (plugin-only)
    → Needs configuration help
    → WebReadyNow offers: Plugin + Setup
    → Pricing workflow is complex
    → WebReadyNow offers: B2B Pricing Implementation (full service)
```
