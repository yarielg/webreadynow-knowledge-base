# WRN Store Monitor — Product File

**Purpose:** Load this file when working on WRN Store Monitor code, marketing, documentation, or the Managed Monitoring service. Cross-reference `03-safe-claims.md` before writing any public-facing copy.

---

## Product Identity

**Commercial structure (decided 2026-07-01 — see `09-agent-outputs/product-alignment/wrn-store-monitor-free-pro-commercial-structure.md`):** Two standalone plugins, not one plugin with a paywall.

| | WRN Store Monitor (Free) | WRN Store Monitor Pro |
|---|---|---|
| **Type** | WooCommerce basic detection/alerting plugin | WooCommerce monitoring and operational intelligence plugin |
| **Current version** | Not yet built — TODO | 1.17.6 (implemented, tested; becomes Pro as-is) |
| **Distribution** | WordPress.org, self-serve, no license key | webreadynow.com, license-gated via WRN Hub |
| **Role** | Acquisition + basic WooCommerce detection/alerting | Monetization + diagnosis + advanced monitoring |
| **Requires the other plugin?** | No | No — standalone, not an add-on |
| **AI** | None | AI Advisor, AI Diagnosis, Code Lab (plan-gated) |

**Module:** Module 2 of the WebReadyNow WooCommerce Solution Suite (Pro)

---

## Free vs. Pro Feature Split

Decided 2026-07-01. Full rationale in `09-agent-outputs/product-alignment/wrn-store-monitor-free-pro-commercial-structure.md`.

**Principle:** Free = passive detection and alerting only — needs no WebReadyNow infrastructure to run forever, satisfies WordPress.org's "genuinely useful, not a crippled trial" requirement. Pro = active/continuous monitoring, reliability infrastructure, and anything involving AI — i.e. everything that costs WebReadyNow engineering upkeep or API spend.

| Capability | Free (WP.org) | Pro (webreadynow.com) |
|---|---|---|
| Failed / stuck order detection | Yes | Yes |
| Payment failure pattern detection | Yes | Yes |
| Inventory / stock issue detection | Yes | Yes |
| Environment diagnostics (PHP/WC/WP/SSL) | Yes | Yes |
| HPOS compatibility status | Yes | Yes |
| Email deliverability check | Yes | Yes |
| Basic checkout config check (HTTPS, order-received reachability, on-scan only) | Yes | Yes |
| Health score dashboard (0–100) | Yes | Yes |
| Daily scheduled scan + manual scan | Yes | Yes |
| Event log | Yes (short retention) | Yes (full retention, higher daily caps) |
| Email alerts | Yes | Yes |
| Silent gateway failure / duplicate charge risk / missing payment metadata / stuck payment candidate monitors | No | Yes — license-gated |
| Customer risk scoring | No | Yes — license-gated |
| WooCommerce Subscriptions monitoring | No | Yes — license-gated |
| Continuous checkout probe, cart probe, wc-ajax probe, response-time trending | No | Yes — license-gated |
| Slack/webhook alerts, Alert History, alert delivery logging, failed-alert notice, re-scan tools | No | Yes — license-gated |
| Incident Timeline | No | Yes — license-gated |
| Diagnostic Payload Builder + Diagnostic Export (JSON) | No | Yes — license-gated |
| AI Advisor (Haiku, monitor-level) | No | Yes — license-gated + AI-plan-gated |
| AI Diagnosis (Sonnet, wizard + Deepen Evidence + 9 evidence tools) | No | Yes — license-gated + AI-plan-gated |
| Code Lab (AI snippet generator, beta) | No | Yes — license-gated + AI-plan-gated |
| Response Log | No | Yes — license-gated |
| Store priority routing / managed-service handoff panel | No | Yes — license-gated, active on Pro — Managed |

**License gating rule:** all Pro-only rows require `WRNSM_License::is_valid()` to be true. The three AI-cost rows (AI Advisor, AI Diagnosis, Code Lab) additionally require an AI-plan gate independent of license validity: valid license **plus** either a BYOK Anthropic key present (Pro — Annual) or WRN-managed AI credits available (Pro — Monthly / Pro — Managed, once metering exists).

**Enforcement status (as of v1.17.5):** every row in this table is now actually code-enforced, not just documented intent. v1.17.4 enforced most of the table (monitors, Alert History UI, Diagnostic Export, AI features, tab locking). v1.17.5 closed the remaining gap: Slack/webhook alert dispatch, continuous checkout probe scheduling/execution, and the failed-alert notice banner were still firing regardless of license until v1.17.5 added the missing `WRNSM_License::is_valid()` checks to those three specific code paths.

**Free/Pro coexistence:** if Free is active while Pro is active, Pro shows a persistent admin notice offering a one-click "Deactivate Free version" action (nonce + `activate_plugins` capability check). Pro must never auto-deactivate Free without explicit user action. Free and Pro use separate DB table prefixes, option names, and cron hook names so simultaneous activation cannot collide or fatal.

---

## What It Does (Verified Capabilities)

**Note:** the capabilities below describe the current codebase, which becomes **WRN Store Monitor Pro** in full. See the Free vs. Pro Feature Split table above for which of these ship in the Free WordPress.org plugin (not yet built) versus which are license-gated Pro-only.

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
- `is_known(id)` — used by checkout monitor and Tool 1/3 to detect non-payment methods in gateway list

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
- Detects unrecognized gateway IDs via `WRNSM_Gateway_Registry::is_known()`
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

Locked 2026-07-01 — see `09-agent-outputs/product-alignment/wrn-store-monitor-free-pro-commercial-structure.md` for full decision record.

| Model | What's Included | Status |
|---|---|---|
| WRN Store Monitor (Free) | WordPress.org install, no license key, no AI | TODO — plugin not yet built |
| WRN Store Monitor Pro — Annual | Pro plugin + license (WRN Hub) + BYOK Anthropic key | **First sellable SKU** — requires only license gating (v1.17.4), no new AI infra |
| WRN Store Monitor Pro — Monthly | Pro plugin + license + WebReadyNow-managed AI usage allowance | Blocked — needs WRN Proxy API / AI credit metering |
| WRN Store Monitor Pro — Managed | Pro — Monthly plan + Managed WooCommerce Monitoring (monthly WebReadyNow alert review and diagnosis) | Blocked — needs WRN Proxy API / AI metering AND a defined Managed Monitoring SLA |

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

### v1.17.6 — License Validation Rejection Handling — ✓ DONE, ✓ COMMITTED (2026-07-01, plugin repo commit `41502fb`)
- Fixed a WRN Hub compatibility bug found during the Pro Annual (BYOK) launch-readiness audit: `WRNSM_License::validate_with_server()` treated any non-200 HTTP response as "server unreachable" without ever reading the response body. WRN Hub returns HTTP 403 for real license rejection states (invalid, expired, suspended, revoked, not activated) — so a genuine rejection was misclassified as a temporary outage and could incorrectly fall back to the 7-day grace period.
- Validation now parses the JSON body regardless of HTTP status. A body with a `success` key is treated as Hub's authoritative answer no matter the status code: `success:true` remains valid (unchanged); `success:false` is invalid immediately, with **no grace period** — this is a real rejection, not an outage.
- Specific Hub rejection reasons (`expired`, `suspended`, `revoked`, `not_activated`, ...) are now preserved in the stored license status instead of being collapsed into a generic `invalid`.
- Network failures, WP_Error responses, and malformed/unparseable bodies (no `success` key at all) still count as "no real answer" — existing grace-period behavior is unchanged for genuine outages.
- `activate()`, `deactivate()`, and the test-key path are untouched — the fix is scoped entirely to `validate_with_server()`.
- Tested via 7 mocked HTTP response scenarios (all pass): 200+success, 403+expired/suspended/revoked/not_activated, WP_Error, malformed JSON. The v1.17.5 same-process license-cache fix was reconfirmed unaffected.
- Version bumped 1.17.5 → 1.17.6; `PROJECT_NOTES.md` updated.

### v1.17.5 — Launch UX and Notification Reliability — ✓ DONE, ✓ COMMITTED (2026-07-01, plugin repo commit `a63bb86`)
- **Notification boundary fixes** — closed a gap where v1.17.4 documented Slack alerts, the continuous checkout probe, and the failed-alert notice as Pro-only but the code didn't actually enforce it yet:
  - Slack/webhook alert dispatch (`class-wrnsm-alert-manager.php`, 3 call sites) now requires `WRNSM_License::is_valid()`. Email alerts remain unconditional — Free baseline, unchanged.
  - Continuous checkout probe scheduling and execution (`class-wrnsm-checkout-prober.php`) now require a valid license. The basic on-scan checkout config check (HTTPS, order-received reachability) stays Free, untouched.
  - The "Alert delivery problem detected" banner is now license-gated.
- **Navigation UX** — sidebar reorganized into three groups: Monitors and Admin (free tabs only) come first, Pro Features is a separate, always-last group. Locked tabs stay clickable — no JS routing or click interception added — but now show dimmed styling and a lock badge (🔒) instead of looking identical to working tabs.
- **Locked-state copy** — shared `license-locked.php` partial now uses one consistent structure everywhere: title "Unlock WRN Store Monitor Pro", fixed body copy, "Go to Settings" CTA, and a secondary "Already entered a license? Refresh this page or run a license check." line. Mixed-tab locks (Activity, Reports) keep their one-line context sentence but follow the same structure.
- **License cache correctness** — `WRNSM_License::activate()`/`activate_test_key()`/`deactivate()` now flush the per-request `is_valid()` cache on state change, so license changes are reflected immediately within the same PHP process instead of only after a fresh request. Surfaced by testing the new checkout-probe gating (reads `is_valid()` during `init`, earlier than any prior call site) — confirmed via same-process WP-CLI test (activate → `is_valid()` YES → deactivate → `is_valid()` NO, no restart needed).
- Live-smoke-tested: unlicensed and licensed states, real Slack webhook delivery (posted to the store's actual configured webhook), real checkout-probe HTTP execution via WP-CLI, full browser verification of nav/copy/gating in both states.
- No new monitors, no Free plugin, no WRN Proxy API, no v1.18.0 scope touched, license gating architecture unchanged from v1.17.4.

### v1.17.4 — Pro Packaging and License Gating — ✓ DONE, ✓ COMMITTED (2026-07-01, plugin repo commit `c0bb75e`)
- `WRNSM_License::ai_available()` added: valid license **and** a way to pay for AI. Today that's BYOK only (Pro — Annual); the WRN-managed-credits branch is stubbed `false` pending v1.19.0/WRN Proxy API.
- Pro-only monitors (`silent_gateway_failure`, `missing_payment_meta`, `duplicate_charge_risk`, `stuck_payment_candidate`, `customer_risk`, `subscriptions`, `store_performance`) stripped from scan results via the existing `wrnsm_monitors` filter when unlicensed — core scan logic untouched.
- 18 server-side gate insertions across admin-post/AJAX handlers: license-only gates on Diagnostic Export, Diagnosis payload build/export, Alert Snapshot/Test Alert tools, Response Notes, Code Lab snippet toggle/delete; AI-cost gates (`ai_available()`) on Incident Summary, AI Diagnosis, per-monitor Diagnose, Global Priority Report, Code Lab generation, follow-up chat, order analysis.
- Locked-state UI: new reusable `admin/views/partials/license-locked.php`. Payment, Subscriptions, Customer Risk, Performance, Diagnosis, and Code Lab tabs show the locked partial when unlicensed. Activity (Incident Timeline + Alert Delivery History locked, Raw Event Log stays free) and Reports (Diagnostic Export locked, CSV/JSON stays free) are gated section-by-section, not whole-tab.
- Free-plugin-detection admin notice + one-click, nonce-protected "Deactivate Free version" action — never auto-deactivates. Basename is a documented placeholder pending the actual Free plugin repo.
- Overview AI Priority Report made collapsible (was taking the full page by default) — added to this pass as a small UX fix, doesn't change commercial scope. Known minor follow-up (not blocking): the summary chip can show "Not generated yet" while stale cached content still displays in the expanded body — pre-existing cache logic.
- Confirmed live (real WordPress/WooCommerce, both unlicensed and licensed-via-test-key states): free-tier baseline stays fully functional unlicensed, `payment_failures` correctly surfaces in the unlocked Orders tab despite the Payment tab itself being fully Pro-locked, AI gating correctly distinguishes "no license" vs. "no API key" vs. both, a real AI call succeeds once license+key are both present, Free-plugin notice/deactivate works end-to-end.
- Explicitly out of scope, not built in this pass: the Free WordPress.org plugin itself, the WRN Proxy API / AI credit metering. Did not move to v1.18.0.

### v1.17.3 — Release Candidate Readiness & UX Positioning — ✓ DONE, ✓ COMMITTED (2026-07-01, plugin repo commit "v1.17.3 mark Code Labs beta and improve evidence status messaging")
- Code Labs UI marked Beta/Experimental: sidebar label "Code Lab — Beta", updated generate-box description, amber warning above generated code ("Experimental code assistance. Always review, test on staging, and back up your site before using generated snippets."). Feature not removed, not marketed as automatic fixing, no follow-up/context memory added.
- Deep Evidence honesty fix: `handle_gather_evidence()` now returns `tools_total`/`tools_succeeded`/`tools_failed`; the Deepen Evidence success message no longer claims a static "9 diagnostic tools" — it now reflects actual tool outcomes (e.g. "7 of 9 diagnostic tools (2 unavailable)"), closing a gap left by v1.17.1's tool fault isolation.
- Reviewed release-facing copy (Overview, Diagnosis, Code Labs) and AI empty/error states against the product promise (detect → AI-assisted diagnosis → evidence-aware explanation → prioritized next actions) — no other copy changes needed.
- Removed three untracked scratch files from the plugin repo (`scripts/test-shot.js`, `scripts/package-lock.json`, `scripts/delete-transients.php`) — dev/ops scaffolding that wasn't gitignored and shouldn't ship in a release zip.
- No new monitors, no service sales copy added, Code Labs not positioned as the main product promise, no move to v1.18.0.

### v1.17.2 — Centralized AI Plumbing and Safety Alignment — ✓ DONE, ✓ COMMITTED (2026-07-01, plugin repo commit `e79334e`)
- `WRNSM_AI_Client` — single shared Anthropic API call, replacing 7 independent `wp_remote_post()` implementations across `AI_Advisor`, `AI_Diagnosis`, `AI_Snippet_Generator`
- `WRNSM_AI_Response_Parser` — single shared 4-tier JSON salvage parser, replacing 4 independent parsers
- `WRNSM_AI_Tools::run_db_query()` retired — removed from `get_definitions()` and `execute()` dispatch; AI can no longer author free-form SQL in the follow-up chat. Implementation kept (commented, unreachable) for reference pending a possible narrow-tool replacement in v1.17.3
- Task types standardized at the client layer: `priority_report`, `incident_summary` (newly wired into feedback — previously had none), `monitor_diagnosis`, `deep_diagnosis`, `diagnosis`, `snippet_generation`
- `WRNSM_AI_Feedback` cost/tooltip now model-aware (`MODEL_PRICING` map) instead of hardcoded to Haiku pricing
- Code Labs generation routed through the shared client/parser; no `snippet_followup` feature added (recommended for v1.17.3 along with richer snippet context)
- See `PROJECT_NOTES.md` in the plugin repo for full file-by-file detail

### v1.17.1 — Evidence Intelligence Stabilization — ✓ DONE, ✓ COMMITTED (2026-07-01, plugin repo commit `8424862`)
- Deepen Evidence panel no longer renders raw tool output (order IDs, error text, AS counts) to the store owner — replaced with a plain tool-count confirmation, matching the "AI input, not user-facing" architecture rule
- `generate_enriched()` truncation bug fixed — base payload cap now sized around the evidence section so the full 20,000-char enriched budget is used, not silently capped at 12,000
- `WRNSM_Evidence_Tools::gather_all()` now fault-isolates each of the 9 tools individually — one failing tool degrades gracefully instead of aborting the whole evidence-gathering request

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

1. **License enforcement** — ✓ implemented in v1.17.4, ✓ closed the Slack/probe/failed-notice enforcement gap in v1.17.5, ✓ fixed WRN Hub 403-rejection handling in v1.17.6 (all committed, live-smoke-tested / mock-tested). The plugin-side code is done. What remains is WRN Hub server-side readiness, not plugin code:
   - WRN Hub product record for slug `wrn-store-monitor` does not exist yet
   - WooCommerce Annual BYOK product/subscription for sale does not exist yet
   - Clean release ZIP/package workflow still needs to be built and confirmed — the current repo ships dev/internal files (`dev-run-tests.php`, `.idea/`, internal docs) in a plain `git archive` build (see Pro Annual BYOK launch-readiness audit)
   - A live activation/validation/deactivation/update/download round-trip against real WRN Hub endpoints is still untested — the v1.17.6 fix was verified with mocked HTTP responses only, since this codebase can't reach the live Hub
   - Version, download URL, and changelog must be seeded on Hub matching the actual shipped version (currently 1.17.6)
   - Price points and WRN Hub license/plan objects for the three Pro SKUs (see decision record)
2. **WRN Proxy API** — replace direct Anthropic key with WRN-managed proxy (usage caps per license tier); blocks Pro — Monthly and Pro — Managed; not required for Pro — Annual (BYOK)
3. **Landing page copy** — product page for webreadynow.com; scope against the three Pro plans (Annual/Monthly/Managed), not a single flat offer
4. **WRN Hub docs seeder** — needs full rewrite to reflect v1.15.0–1.17.0 features
5. **Free plugin** — does not exist yet; separate repo and version track (own `v1.0.0`), required before any WordPress.org listing
6. **Managed Monitoring SLA** — not yet defined; blocks Pro — Managed specifically

## Next Sprint Roadmap

**v1.17.2 through v1.17.6 are all committed.** License enforcement (v1.17.4), the Slack/probe/failed-notice enforcement gap (v1.17.5), and WRN Hub 403-rejection handling (v1.17.6) are all closed on the plugin-code side — see `09-agent-outputs/product-alignment/wrn-store-monitor-free-pro-commercial-structure.md` for the full commercial decision. Do not move to v1.18.0 (Sprint B) without a fresh scoping pass; v1.18.0 stays scoped to Sprint B as already planned below unless that pass changes it. The Free WordPress.org plugin is a **separate repository and version track**, starting at its own `v1.0.0` — it does not consume Pro's version numbers.

| Sprint | Version | Goal |
|---|---|---|
| — | v1.17.2 | Centralized AI Plumbing and Safety Alignment — ✓ committed |
| — | v1.17.3 | Release Candidate Readiness & UX Positioning — ✓ committed |
| — | v1.17.4 | Pro Packaging and License Gating — ✓ committed (`c0bb75e`) |
| — | v1.17.5 | Launch UX and Notification Reliability — ✓ committed (`a63bb86`) |
| — | v1.17.6 | License Validation Rejection Handling — ✓ committed (`41502fb`) |
| Sprint B | v1.18.0 (Pro) | Evidence UX — gateway health panel, guided investigation wizard, timeline correlation, plugin conflict indicator, version checker |
| Sprint C | v1.19.0 (Pro) | Managed service handoff — escalation panel, evidence summary export, WRN Proxy AI, unblocks Pro — Monthly and Pro — Managed |
| — | v1.0.0 (Free, separate repo) | Standalone WordPress.org Free plugin — TODO, not yet scoped |

**Next step is not a version number — it's launch ops:** `WRN Hub Product Setup + Clean Release ZIP + Live License/Update Test`. All plugin-side license/notification/UX code needed for Pro Annual (BYOK) is done as of v1.17.6. What's left is server-side (WRN Hub product/plan/price records, seeded version/changelog/download) and packaging (a clean release ZIP workflow, then a real activation/validation/deactivation/update/download round-trip against live Hub) — see the Pre-Release Blockers list above for the itemized gaps. This is explicitly **not** more monitor/feature work and **not** product-page copy yet — both come after this phase.

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
