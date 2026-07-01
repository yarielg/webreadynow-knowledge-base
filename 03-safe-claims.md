# WebReadyNow — Safe Claims

**Purpose:** Read this file before writing any product copy, service page, marketing content, case study, or documentation. This is the claim authority. If something is not listed as Safe here, it is not safe to publish without a new capability audit.

---

## Global Rule

Do not use unsupported statistics, exaggerated claims, or product promises that are not verified by codebase audit, service process, or case study proof.

---

## Safe Language Patterns (All Products and Services)

Use these patterns freely:

- Helps detect
- Surfaces
- Alerts when thresholds are crossed
- Logs events
- Supports diagnosis
- Provides diagnostic context
- Helps identify patterns
- Helps organize evidence
- Reduces time to understand
- Assists with triage

---

## Unsafe Language (All Products and Services)

Never use without full verification:

- Automatically fixes
- Guarantees recovery
- Prevents all revenue loss
- Detects every issue
- Monitors the full journey
- Replaces human support
- Complete protection
- Full coverage
- Always catches

---

## WRN Store Monitor — Safe Claims

### Safe to claim today

- WRN Store Monitor helps detect WooCommerce operational issues earlier.
- WRN Store Monitor surfaces failed orders, stuck orders, payment failure patterns, checkout errors, and store health signals.
- WRN Store Monitor logs important events with timestamps.
- WRN Store Monitor sends email and Slack alerts when configured thresholds are crossed.
- WRN Store Monitor records alert delivery attempts so store owners can verify what was sent and when.
- WRN Store Monitor provides an AI advisor when an Anthropic API key is configured.
- WRN Store Monitor helps organize diagnostic evidence for faster troubleshooting.
- WRN Store Monitor monitors key WooCommerce checkout health signals: checkout page availability, checkout response time, WooCommerce AJAX health, cart page availability, and order confirmation page availability.

**Added in v1.14.0 — Diagnostic Evidence Layer:**
- WRN Store Monitor records diagnostic events inside WooCommerce — including order status changes, payment failures, checkout errors, scan results, plugin activations, and gateway setting changes.
- WRN Store Monitor logs plugin activation and deactivation events so the store history includes environment changes around operational issues.
- WRN Store Monitor logs payment gateway enabled and disabled changes so timeline evidence includes gateway state at the time of any failure.
- WRN Store Monitor stores a compact snapshot of scan evidence on every scan, including per-monitor severity, issue counts, and an environment fingerprint (PHP version, WooCommerce version, WordPress version, active gateways, HPOS status).
- WRN Store Monitor records what triggered each alert — the specific monitors, health score, and threshold values — so alert history includes diagnostic context, not just delivery status.
- WRN Store Monitor records missing diagnostic data observations after every scan: no recent orders, no successful payments in 72h, no enabled gateways, auto-scan not scheduled, and WP-Cron overdue signals.
- WRN Store Monitor shows all of these records in the Incident Timeline tab alongside scan history, payment events, checkout errors, and alert deliveries.

### Safe with careful wording

| Claim | Safer Wording |
|---|---|
| "Monitors checkout" | Monitors checkout configuration and captures checkout error messages displayed to customers |
| "Detects checkout failures" | Captures checkout errors that WooCommerce displays to customers and alerts when error patterns cross thresholds |
| "AI diagnosis" | AI-assisted triage using available store data; human verification required |
| "Alert delivery" | Records alert attempts and delivery status; does not guarantee inbox placement or Slack delivery |
| "Prepares evidence for AI diagnosis" | Captures and stores structured WooCommerce-layer evidence that can support future AI-assisted triage; does not send data to AI automatically |
| "Helps WebReadyNow investigate faster" | Preserves observable WooCommerce-layer evidence and missing-data signals that reduce investigation time when WebReadyNow reviews a store |

### Unsafe — do not claim

- Automatically fixes checkout failures
- Prevents all revenue loss
- Guarantees checkout will never break
- Detects every possible WooCommerce issue
- Monitors the full checkout journey
- Performs synthetic checkout testing
- Simulates real payment submission
- Guarantees revenue recovery
- Replaces human WooCommerce support
- Guaranteed alert delivery
- Guaranteed inbox placement
- AI diagnosis is complete or confirms root cause
- Diagnoses root cause without human investigation
- Automatic remediation based on diagnostic evidence
- Monitors server-level or infrastructure-level issues
- Investigates gateway dashboards or payment processor accounts
- Has WRN Hub integration for AI diagnosis (not yet built)

---

## WRN Hub — Safe Claims

**Note:** WRN Hub is internal infrastructure installed only on webreadynow.com. It is not a standalone product. Claims below describe what WebReadyNow can say about its licensing and documentation systems — not about a product customers install.

### Safe to claim

- WebReadyNow uses WRN Hub to manage licenses, updates, downloads, and documentation for its plugins.
- Customers can view and copy their license keys from the WebReadyNow My Account area.
- Active license holders can download their plugin from My Account.
- WebReadyNow plugin documentation is available publicly at webreadynow.com/docs.
- Documentation is searchable.
- Subscription-based license status can sync with WooCommerce Subscriptions when configured.

### Unsafe — do not claim

- WRN Hub is available for customers to install.
- WRN Hub is a standalone product.
- License keys are emailed automatically after purchase.
- Downloads are fully protected against direct URL sharing.
- License keys use cryptographically secure random generation.
- WRN Hub provides customer support ticketing.

---

## WRN Pricing Rules — Safe Claims

- Supports wholesale pricing in WooCommerce
- Supports role-based pricing
- Supports B2B/B2C pricing separation
- Supports customer-specific pricing and price lists
- Supports VIP and distributor pricing workflows

### Unsafe until verified
- TODO: Audit codebase to confirm exact supported pricing rule types and edge cases before writing detailed product copy

---

## WRN Pricing Rules Pro — WRN Hub License Integration

**Source:** WRN Pricing Rules Pro v1.2.1 codebase audit + post-fix verification, 2026-06-29.

### Safe to claim

- WRN Pricing Rules Pro connects to the WebReadyNow license server for activation and ongoing validation.
- Entering a valid license key activates Pro features on the site immediately.
- License validity is automatically re-checked daily when WP-Cron is functioning.
- A 7-day grace period keeps Pro features active if the WebReadyNow license server is temporarily unreachable.
- Pro features are gated behind license validation and stop working when the license is confirmed invalid.
- Updates appear in the standard WordPress plugin updates screen.
- Plugin updates are downloaded through a license-authenticated WRN Hub endpoint.
- Customers can manage their subscription and license key from their WebReadyNow account at webreadynow.com/my-account.
- Suspended, expired, revoked, and invalid license states display customer-facing guidance on the Pro License admin page and in an admin notice on WRN Pricing admin pages.

### Safe with wording limits

| Claim | Approved wording |
|---|---|
| Deactivation frees the site slot | "Deactivating the license frees the site slot when the WebReadyNow license server is reachable at that moment." |
| One-click updates | "Updates install with one click through the standard WordPress update system when the license is active and webreadynow.com is reachable." |

### Unsafe — do not claim

- License keys are emailed automatically after purchase. *(Keys are retrieved from webreadynow.com/my-account, not email.)*
- Deactivation always frees the site slot. *(Fire-and-forget — slot cleanup depends on server reachability at deactivation time.)*
- License status messages cover every possible server or billing situation.
- Updates work without a stored license key. *(Download URL requires the key; update check is public but download will fail without a valid key.)*
- The plugin can update if outbound HTTPS requests to webreadynow.com are blocked.
- Pro features remain available indefinitely without a valid license.

---

## WRN Pricing Rules Pro — Feature Claims

**Source:** WRN Pricing Rules Pro v1.2.1 codebase audit, 2026-06-29. See `09-agent-outputs/codebase-audits/wr-price-list-pro-feature-capability-audit-v1.2.1.md`.

### Safe to claim

- WRN Pricing Rules Pro supports advanced discount rules based on role, customer, product, category, quantity, cart totals, cart quantity, first order status, and past order count.
- Store owners can create percent-off or fixed-amount discounts.
- Discounts can be applied per item or as a single cart-line fee.
- Rules can use first-match or stack-all behavior.
- Stacked discounts can use additive or sequential math.
- Store owners can control whether coupons stack with Pro discount rules.
- WRN Pricing Rules Pro supports volume pricing tables built from discount rules.
- Volume pricing tables can be displayed on product pages with configurable placement, and update automatically when a customer selects a variation.
- WRN Pricing Rules Pro includes a `[wrn_price_table]` shortcode with filtering, column control, pagination, and a built-in shortcode generator.
- Individual customers can be assigned to specific price lists.
- Customer-specific price list assignment overrides role-based pricing.
- WRN Pricing Rules Pro includes a Pricing Inspector for reviewing product and cart pricing context in the WordPress admin bar.
- Store owners can configure expiry notifications for price lists and discount rules.
- WRN Pricing Rules Pro is compatible with WooCommerce HPOS.
- Discount rules appear as labeled lines in the cart and order totals, and are recorded in the order note.

### Safe with wording limits

| Claim | Approved wording |
|---|---|
| Cart-total rules with product/category conditions | "Cart-total rules scoped to specific products or categories apply the discount to the total of matching items, not the full cart total." |
| Customer search scope | "Customers can be searched and assigned by name or email; search returns up to 20 results at a time." |
| Pricing Inspector cross-theme support | "The Pricing Inspector works on the storefront admin bar; minicart refresh behavior may need theme-specific adaptation on stores not tested by WebReadyNow." |
| Expiry notifications | "Expiry notifications cover price lists and discount rules. License renewal notifications are not included." |
| Order count conditions | "First-order and past-order-count conditions count completed orders only. Pending and processing orders do not count." |

### Unsafe — do not claim

- Unlimited customer search.
- License expiry notifications are included in the plugin.
- The Pricing Inspector works perfectly on every WooCommerce theme without any adaptation.
- Cart-total rules with product or category conditions apply to the entire cart total.
- All pricing behavior is automatic with no setup required.
- The plugin replaces custom B2B pricing setup or store-specific configuration work.
- Every pricing scenario is supported without custom development.

---

## WebReadyNow Services — Safe Claims

### WooCommerce Store Audit
- WebReadyNow diagnoses WooCommerce problems before rebuilding, adding plugins, or spending money guessing.

### Performance and Checkout Rescue
- WebReadyNow investigates and fixes slow WooCommerce stores, checkout failures, payment and order issues, caching problems, and performance bottlenecks.
- Do not promise fixed timelines or guaranteed uptime improvements without scoping the engagement.

### WooCommerce Integrations
- WebReadyNow connects WooCommerce with ERP, inventory, ShipStation, QuickBooks, SAP Business One, fulfillment systems, and external platforms.
- Do not guarantee integration success without a technical discovery.

### Custom Plugin Development
- WebReadyNow builds custom WooCommerce functionality when off-the-shelf plugins create risk, complexity, or poor fit.

### Managed WooCommerce Monitoring
- WebReadyNow reviews WRN Store Monitor alerts, diagnoses issues, and supports store reliability.
- Do not claim guaranteed response times without a defined SLA in the engagement.

---

## Proof-Backed Claims

These claims are backed by published case studies or confirmed client work. Full details in `08-proof/case-study-index.md`.

- B2B WooCommerce with custom price list plugin (WR Price List Manager case study — published)
- SAP Business One integration with WooCommerce (Quest Technology International)
- WooCommerce performance and operations rescue (Samurai Fireworks)
- WooCommerce Subscriptions and Stripe payment failure recovery (EarthSavers)
- High-volume WooCommerce with subscriptions, ERP, ShipStation (Bee-Och)

Do not exaggerate outcomes from these case studies. Use only what is confirmed.
