# WebReadyNow — Offer Map

**Purpose:** Read this file when making decisions about product direction, service packaging, CTAs, pricing models, or business development. It maps every current offer and shows how products and services connect.

---

## Offer Model Matrix

| Offer | Type | Delivery | Current Status |
|---|---|---|---|
| WRN Store Monitor — Plugin only | Product | Self-serve | Active — pre-release |
| WRN Store Monitor — Plugin + Setup | Productized service | Plugin + onboarding session | TODO — define scope |
| WRN Store Monitor — Managed Monitoring | Recurring service | Plugin + human monthly review | TODO — define SLA |
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

### WRN Store Monitor

**What it is:** WooCommerce monitoring and operational intelligence plugin.

**Current version:** 1.11.0

**Offer models:**
1. Plugin-only — self-serve purchase
2. Plugin + Setup — plugin license plus configuration and onboarding
3. Plugin + Managed Monitoring — plugin plus WebReadyNow monthly monitoring service

**Who buys it:**
- WooCommerce store owners who want early warning on operational issues
- Agencies managing WooCommerce stores for clients
- Stores with meaningful revenue at risk from checkout or payment failures

**Key value:**
- Detect failed orders, stuck orders, payment anomalies
- Capture checkout errors in real time
- Alert via email and Slack
- AI-assisted triage (requires Anthropic API key)
- Health score and operational dashboard

**Primary CTA:** Install WRN Store Monitor / Get Managed Monitoring

**Connects to service:** Managed WooCommerce Monitoring — when issues require diagnosis, root-cause repair, or ongoing expert support

Full details in `06-products/wrn-store-monitor.md`.

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
Store owner discovers WRN Store Monitor
    → Installs plugin (plugin-only)
    → Sees an alert they can't diagnose
    → WebReadyNow offers: Store Audit or Managed Monitoring
        → Audit reveals deeper issues
        → Rescue or Integration engagement
        → Custom plugin if needed
        → Managed Monitoring retainer
```

```
B2B store needs wholesale pricing
    → Buys WRN Pricing Rules Pro (plugin-only)
    → Needs configuration help
    → WebReadyNow offers: Plugin + Setup
    → Pricing workflow is complex
    → WebReadyNow offers: B2B Pricing Implementation (full service)
```
