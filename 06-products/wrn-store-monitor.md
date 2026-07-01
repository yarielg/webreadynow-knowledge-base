# WRN Store Monitor — Product File

**Purpose:** Load this file when working on WRN Store Monitor code, marketing, documentation, or the Managed Monitoring service. Cross-reference `03-safe-claims.md` before writing any public-facing copy.

---

## Product Identity

**Name:** WRN Store Monitor
**Type:** WooCommerce monitoring and operational intelligence plugin
**Current version:** 1.17.0
**Distribution:** Paid plugin via WRN Hub (webreadynow.com). No WP.org free version.
**Module:** Module 2 of the WebReadyNow WooCommerce Solution Suite

---

## What It Does (Verified Capabilities)

### Operations Dashboard
- WooCommerce health overview with severity scoring (Good / Notice / Warning / Critical)
- Health score (0–100) based on all monitor results
- Scan history sparkline and 30-day trend panel
- Managed service handoff panel (appears when health < 70 or critical monitors exist)

### Monitors
- Failed order detection
- Stuck order detection
- Payment failure patterns
- Silent gateway failure signals
- Duplicate charge risk detection
- Missing payment metadata detection
- Stuck payment candidate detection
- Checkout configuration checks (HTTPS, order-received endpoint)
- Checkout page response time (during scan)
- Cart page HTTP probe
- WooCommerce AJAX (wc-ajax) probe
- Continuous checkout probe via Action Scheduler
- Action Scheduler health
- WP-Cron risk
- Inventory/stock issues
- Environment diagnostics (PHP, WC version, SSL, object cache, debug.log size)
- HPOS compatibility status
- Email deliverability (SMTP plugin, from address, critical WC email status)
- Customer risk scoring
- WooCommerce Subscriptions monitoring (requires WC Subscriptions)
- Store performance (checkout error capture and pattern analysis)

### Event Logging
- Structured event log with timestamps
- Daily caps per event type
- Checkout error capture (classic and Blocks checkout)
- Pattern-based severity (same error 3x = warning, 10x = critical)
- Plugin lifecycle events (activated/deactivated) — admin-only, daily cap 20
- Gateway change events (enabled/disabled) — fires on settings save and AJAX toggle, daily cap 50

### Alerting
- Email alerts when monitors escalate to Critical
- Slack/webhook alerts
- Real-time checkout error spike alerts (fires when threshold crossed without waiting for scan)
- Continuous checkout probe alerts (on ok → failing transition, with recovery notice)
- Alert delivery logging (channel, status, error, timestamp)
- Alert History admin tab
- Failed alert notice in dashboard
- Reset Alert Snapshot tool
- Send Test Alert tool
- Store priority routing: High/Critical stores prepend priority tag to alert email subjects

### Diagnosis & Export
- Incident Timeline tab — chronological feed of scans, events, alerts, plugin lifecycle changes, gateway changes, and missing data signals (last 7 days)
- Diagnostic Export — comprehensive JSON download (scan results, event log, alert history, probe states, environment)
- AI Incident Summary — plain-English summary of active critical issues with business impact
- Response Log — timestamped notes (disabled by default, opt-in in Settings)

### AI Diagnosis System (v1.16.0–1.17.0)

**Diagnosis tab** — single-flow wizard: describe symptom → AI analysis → Deepen Evidence (optional second pass)

**`WRNSM_AI_Diagnosis`** — full structured diagnosis using Claude Sonnet 4.6 (independent model from monitor AI which uses Haiku)
- `generate(payload_row, api_key)` — standard diagnosis: 12,000 char payload cap, `prompt_version 1.1`, stores in `wrnsm_ai_diagnoses`
- `generate_enriched(payload_row, api_key)` — second pass: gathers all 9 evidence tools, builds enriched message (20,000 char cap), `prompt_version 1.2`, `trigger_type = evidence_enriched`
- 13-key JSON schema: `problem_classification`, `likely_contributing_factors`, `evidence_used`, `missing_data_impact`, `safe_to_conclude`, `suggested_only`, `cannot_determine`, `recommended_next_steps`, `human_investigation_required`, `escalation_reason`, `plain_english_summary`, `technical_summary`, `ai_limitations_note`
- Rate limit: standard pass 300 seconds per user; enriched pass 30 seconds (separate transient — prevents double-click only)
- HTTP timeout: 120 seconds with `http_request_timeout` filter override (prevents WP capping it)
- `render()` output: recommended next steps rendered as a numbered vertical timeline (not 3-column grid) — Step 1 = Store admin (blue), Step 2 = Developer/WRN (purple), Step 3 = Hosting/Gateway (orange); connected by a vertical CSS line between numbered circles

**Deepen Evidence panel** (v1.17.0) — appears in results panel after any diagnosis
- Button triggers `wrnsm_gather_evidence` AJAX action
- Calls all 9 `WRNSM_Evidence_Tools` methods
- Feeds structured tool output into enriched Sonnet pass
- Replaces AI result with enriched diagnosis; raw evidence output is intentionally hidden (it is input for the AI, not user-facing output)
- After enriched update: flash-highlight animation (`is-updated` CSS class) + smooth scroll to result card so the update is impossible to miss
- Confirmation note: "↑ AI diagnosis updated with live evidence from 9 diagnostic tools." appears below the button

**`WRNSM_Evidence_Tools`** — 9 controlled read-only diagnostic tools (no arbitrary SQL, no PII)
1. `get_checkout_type_context()` — checkout type (block/shortcode), available vs registered gateway IDs, non-payment methods in list, HTTPS
2. `get_recent_checkout_error_clusters(7d)` — grouped checkout errors with counts and associated gateways (top 5)
3. `get_invalid_gateway_id_summary()` — suspicious gateway registrations (empty IDs, non-payment classes)
4. `get_affected_orders_summary(30d)` — order status counts, stuck count, gateway distribution of stuck orders
5. `get_gateway_failure_summary(7d)` — per-gateway success rate via `WRNSM_Payment_Health`
6. `get_action_scheduler_gateway_context()` — AS job counts by status for payment-related hooks
7. `get_slow_request_summary()` — last checkout probe result + trend direction from scan history
8. `get_plugins_touching_checkout_context()` — plugin folders registered on 7 checkout/payment hooks via PHP Reflection; flags non-gateway plugins on `woocommerce_payment_gateways`
9. `get_order_notice_summary(7d, 50 orders)` — 13 regex patterns matched against order notes (counts only — no raw text)

**`WRNSM_Gateway_Registry`** — static registry of ~45 known payment gateway IDs
- Covers: WooPayments, Stripe (official + Payment Plugins), PayPal PPCP, Square, Authorize.Net, Braintree, Amazon Pay, Klarna, Affirm, Afterpay, BACS, COD, Cheque
- `is_known_payment_gateway(id)` — used by checkout monitor and Tool 1/3 to detect non-payment methods in gateway list

**Diagnostic Payload (v1.16.1–1.16.2)**
- Payload now includes: event aggregates, payment health (7d), checkout probe state, top checkout errors (7d)
- `has_missing_data_declaration` correctly set when `unavailable_at_assembly` key exists
- `confidence_readiness` bug fixed (undefined variable no longer forces `limited` for all categories)
- Schema version: 2.2; char cap: 12,000 (standard), 20,000 (enriched)

**AI system prompt**
- Hypothesis ranking rules: gateway rate data > event aggregates > repeated patterns > plausible-only
- Frontend/JS optimizer hypotheses must stay secondary unless checkout probe or browser evidence supports them

### Checkout Type Detection (v1.16.3)
- `WRNSM_Checkout_Tracker::detect_checkout_type()` — public static; checks `has_block('woocommerce/checkout')` then `has_shortcode('woocommerce_checkout')`; returns `'block'|'shortcode'|'unknown'`
- Checkout monitor stores `checkout_type` in scan meta
- Diagnostic payload includes `checkout_type` in environment section (both snapshot and runtime paths)
- AI user message prints `checkout_type` in ENVIRONMENT block
- Block checkout MutationObserver debounced 600ms + visibility check — eliminates false positives from React hydration transient states

### Checkout Configuration Monitor (v1.17.0 additions)
- Detects unrecognized gateway IDs via `WRNSM_Gateway_Registry::is_known_payment_gateway()`
- Detects non-payment methods (shipping/pickup classes) registered in the payment gateway list
- Fires `warning` when either is found; includes gateway class name in meta
- `invalid_gateway_ids` and `non_payment_in_list` added to monitor meta

### Diagnosis Tab — UI Principles (established 2026-06-30)

- Evidence tool output is **not user-facing** — it is raw material gathered for the AI enriched pass. The Deepen Evidence panel never shows raw AS queues, hook maps, or tool dumps to the store owner.
- "Diagnostic Payload" collapsible section contains only: raw payload JSON viewer + Copy + Download. All duplicate sections (Data Readiness, Flagged Monitors, Store Health) have been removed — that data is already on other tabs and already digested by the AI.
- No WebReadyNow cross-sell copy in the Diagnosis tab until the service handoff approach is decided.
- Design principle: the AI diagnosis card is the primary output. Everything else is secondary or hidden.

### Diagnostic Payload Builder (v1.15.0)
- Incident Context form — stores admin-entered symptom category, problem description, start date, ongoing/intermittent flags, affected gateway, customer-visible error, recent changes in `wrnsm_incident_context` wp_option
- `WRNSM_Diagnostic_Payload::build()` — assembles sanitized diagnostic payload from all evidence: scan snapshot, event log (50 rows), alert log (20 rows), missing data observations, incident context, order status summary (7 days), active plugins, environment
- `wrnsm_diagnostic_payloads` table — stores up to 10 payloads; `trigger_type 'manual'` or `'evidence_enriched'`; capped at 10 rows
- Privacy: amounts excluded; customer names/emails/addresses/phone numbers excluded; API keys/secrets/tokens scrubbed recursively; Stripe key patterns and 16-digit strings scrubbed from values; alert subjects excluded; order notes excluded
- Payload preview: structured sections (health, monitors, missing data, events summary, plugins) + raw JSON with Copy button
- Diagnostic Export (Reports tab) upgraded: now includes `scan_snapshot`, `trigger_context` in alert rows, and `missing_data` section
- Schema version: 2.2

### Diagnostic Evidence Layer (v1.14.0)
- Scan snapshots — compact per-monitor severity/count + environment fingerprint stored on every scan in `wrnsm_scan_history.snapshot_data`
- Alert trigger context — what caused each alert (monitors, health score, thresholds) stored in `wrnsm_alert_log.trigger_context`
- Missing data log — silent absence signals recorded after every scan with 24h dedup; check types: `no_orders_48h`, `no_successful_payment_72h`, `gateway_not_configured`, `no_recent_scan`, `cron_overdue`

### AI Advisor (Monitor-level)
- AI-assisted explanation per monitor using Claude Haiku (independent model from Diagnosis which uses Sonnet)
- Follow-up conversation per monitor
- AI tools: read debug.log, read .htaccess, get WC logs, get active plugins, get store settings, get Action Scheduler jobs, get WC sessions, run SELECT-only DB queries
- Session persistence per user per monitor
- AI Incident Summary for critical states
- **Important constraint:** Monitor-level AI tools are legacy; future evidence access should use controlled `WRNSM_Evidence_Tools` methods only — free-form DB queries in the AI tools layer are a known risk

### Admin Tools
- Diagnostic Export (JSON) — for sending to WebReadyNow for remote diagnosis
- Code Lab (custom PHP snippet runner with AI generation)

---

## What It Does NOT Do

- Automatic issue remediation of any kind
- Synthetic checkout testing (no simulated cart/payment submission)
- Full checkout journey monitoring
- Payment submission simulation
- Detection of every possible WooCommerce failure
- Root cause confirmation (AI provides ranked hypotheses; humans confirm)
- Replacement of human WooCommerce support
- Gateway dashboard access or webhook delivery log inspection
- Server-level or infrastructure monitoring
- Free-form AI database queries (controlled evidence tools only — lesson from Samurai Fireworks live incident 2026-06-30)
- WRN Hub proxy API (deferred — Sprint C / v1.19.0)

---

## Offer Models

| Model | What's Included | Status |
|---|---|---|
| Plugin-only | License + self-serve install | Pre-release |
| Plugin + Setup | License + WebReadyNow configuration session | TODO — define scope |
| Managed Monitoring | License + monthly WebReadyNow alert review and diagnosis | TODO — define SLA |

---

## Roadmap Status

### Sprint 1 — Checkout Health Upgrade (v1.10.0) — ✓ DONE
- Cart page HTTP probe
- WooCommerce AJAX probe
- Order-received endpoint check
- Continuous checkout probe (Action Scheduler)

### Sprint 2 — Alert Reliability Upgrade (v1.11.0) — ✓ DONE
- Alert delivery logging
- Alert History admin tab
- Failed alert notice
- Alert diagnostic panel
- Send Test Alert / Reset Snapshot tools

### Sprint 3 — Diagnosis Support Upgrade (v1.12.0) — ✓ DONE
- Incident timeline tab
- Diagnostic export (JSON)
- AI incident summary
- Managed service handoff panel

### Sprint 4 — Managed Service Layer (v1.13.0) — ✓ DONE
- Store priority routing (Normal / High / Critical)
- Response log (opt-in)
- Removed: client status dropdown and escalation workflow (rethinking service handoff approach)

### v1.17.0 — Controlled Evidence Layer — ✓ DONE
- `WRNSM_Gateway_Registry` — static registry of ~45 known gateway IDs
- `WRNSM_Evidence_Tools` — 9 controlled read-only tools, `gather_all()` aggregate method
- `WRNSM_AI_Diagnosis::generate_enriched()` — second Sonnet pass with tool evidence, `prompt_version 1.2`, 20k char cap
- Checkout monitor: detects invalid gateway IDs and non-payment methods in gateway list
- Diagnosis tab: "Deepen Evidence" panel with `wrnsm_gather_evidence` AJAX action
- Architecture rule established: AI never accesses DB directly; all evidence via named tools

### v1.16.3 — Block Checkout Type Detection — ✓ DONE
- `WRNSM_Checkout_Tracker::detect_checkout_type()` public static
- `checkout_type` in monitor meta, payload environment section, AI user message
- MutationObserver debounce + visibility check — eliminates React hydration false positives

### v1.16.2 — AI Signal Fixes — ✓ DONE
- `has_missing_data_declaration` bug fixed
- Hypothesis ranking rules in system prompt
- Frontend/JS optimizer hypotheses constrained to secondary position

### v1.16.1 — AI Diagnosis Quality Quick Wins — ✓ DONE
- Diagnosis model: `claude-sonnet-4-6` (independent of monitor AI which stays on Haiku)
- Payload enriched: event aggregates, payment health (7d), checkout probe state, top checkout errors
- `http_request_timeout` filter override; char cap 6k→12k; timeout 60s→120s
- Undefined variable bug in `build_payload_quality()` fixed (was forcing `confidence_readiness = limited`)

### v1.16.0 — AI Diagnosis Draft Mode — ✓ DONE
- `WRNSM_AI_Diagnosis` class + `wrnsm_ai_diagnoses` table
- Manual diagnosis trigger from Diagnosis tab wizard
- 13-key JSON schema; rate limit 300s; stores per payload_id

### AI Diagnostic System Sprint 2 — Diagnostic Payload Builder (v1.15.0) — ✓ DONE
- `WRNSM_Incident_Context` — wp_options-backed incident form with 8 fields
- `WRNSM_Diagnostic_Payload` — assembler with recursive scrubber, privacy enforcement
- `WRNSM_Diagnostic_Payload_Store` — wrnsm_diagnostic_payloads table, 10-row cap
- Diagnosis admin tab — single-flow wizard (describe → AI analysis → deepen evidence)
- Reports tab Diagnostic Export upgraded — includes scan_snapshot, trigger_context, missing_data

### AI Diagnostic System Sprint 1 — Diagnostic Evidence Layer (v1.14.0) — ✓ DONE
- `wrnsm_scan_history.snapshot_data` — compact per-monitor severity/count + environment fingerprint stored on every scan
- `wrnsm_alert_log.trigger_context` — records why each alert fired (monitor IDs, health score, error count vs threshold)
- `wrnsm_missing_data_log` new table — silent absence signals with 24h dedup and 30-day retention
- `plugin_lifecycle` and `gateway_change` event types in wrnsm_events
- Missing data checks: no_orders_48h, no_successful_payment_72h, gateway_not_configured, no_recent_scan, cron_overdue
- Timeline tab shows all new entry types alongside existing scan/event/alert entries

---

## Pre-Release Blockers

1. **License enforcement** — `WRNSM_License::is_valid()` is never called; hard blocker before selling
2. **WRN Proxy API** — replace direct Anthropic key with WRN-managed proxy (usage caps per license tier); planned for v1.19.0
3. **Landing page copy** — product page for webreadynow.com
4. **WRN Hub docs seeder** — needs full rewrite to reflect v1.15.0–1.17.0 features

## Next Sprint Roadmap

| Sprint | Version | Goal |
|---|---|---|
| Sprint B | v1.18.0 | Evidence UX — gateway health panel, guided investigation wizard, timeline correlation, plugin conflict indicator, version checker |
| Sprint C | v1.19.0 | Managed service handoff — escalation panel, evidence summary export, WRN Proxy AI, license gate |

---

## Architecture Notes

- PHP 8+, WordPress 6.0+, WooCommerce 7.0+
- HPOS-compatible (declared)
- DB tables: `wrnsm_events`, `wrnsm_scan_history`, `wrnsm_ai_feedback`, `wrnsm_alert_log`, `wrnsm_missing_data_log`, `wrnsm_diagnostic_payloads`, `wrnsm_ai_diagnoses`
- All tables auto-install on `plugins_loaded` via `needs_install()` check
- Action Scheduler for daily scans and continuous probe (group: `wrn-store-monitor`)
- Scan results cached 5 min in `wrnsm_scan_results` transient
- Alert snapshot in `wrnsm_alert_severity_snapshot` option (controls newly-critical detection)

---

## Codebase Location

`C:\wamp64\www\samuraifireworks.com\wp-content\plugins\wrn-store-monitor`
