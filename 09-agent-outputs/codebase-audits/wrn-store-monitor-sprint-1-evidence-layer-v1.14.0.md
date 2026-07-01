# Codebase Capability Audit: WRN Store Monitor — Sprint 1 Diagnostic Evidence Layer

**Version:** 1.13.0 → 1.14.0
**Date:** 2026-06-29
**Scope:** Sprint 1 Diagnostic Evidence Layer — evidence enrichment only; no AI submission, no WRN Hub connection, no marketing copy
**Auditor:** Codebase Capability Auditor Agent

---

## Audit Scope

Files reviewed or modified in this sprint:

- `includes/class-wrnsm-event-logger.php` — event type constants and daily caps
- `includes/class-wrnsm-scan-history.php` — TABLE_VERSION, schema, record() method
- `includes/class-wrnsm-alert-log.php` — DB_VERSION, schema, record() method
- `includes/class-wrnsm-logger-hooks.php` — new hooks and handlers
- `includes/class-wrnsm-alert-manager.php` — trigger_context passed to alert log
- `includes/class-wrnsm-plugin.php` — load order, install, daily scan, scan history call
- `includes/class-wrnsm-missing-data-log.php` — new file, new table
- `admin/views/tabs/tab-timeline.php` — new entry types rendered
- `wrn-store-monitor.php` — version 1.14.0, activation hook

---

## Executive Capability Decision

WRN Store Monitor v1.14.0 completes the Diagnostic Evidence Layer. The plugin now preserves richer structured evidence on every scan, alert, and store-level event — including scan snapshots, alert trigger context, plugin lifecycle changes, gateway state changes, and silent-absence observations. All evidence is stored locally in WordPress database tables. No AI submission, no external transmission, and no diagnosis occurs automatically. The evidence layer is complete; the next capability that changes what can be safely claimed is Sprint 2 (Diagnostic Payload Builder or AI submission layer).

---

## Implemented Capabilities

**Capability:** Scan snapshot storage
**Evidence:** `WRNSM_Scan_History::record()`, `build_snapshot()`, `wrnsm_scan_history.snapshot_data`
**What it actually does:** On every scan, stores a compact JSON object per monitor (severity, detail count, short summary) plus an environment fingerprint (PHP version, WC version, WP version, HPOS state, object cache, active gateways). Does NOT store raw detail arrays.
**What the user sees:** Column available in DB for future AI submission or diagnostic export; not yet surfaced in admin UI.
**Safe claim:** WRN Store Monitor stores a compact snapshot of scan evidence on every scan, including per-monitor severity, issue counts, and an environment fingerprint.

---

**Capability:** Alert trigger context logging
**Evidence:** `WRNSM_Alert_Log::record()`, `wrnsm_alert_log.trigger_context`, `WRNSM_Alert_Manager::on_scan()`
**What it actually does:** When a critical scan alert or checkout spike alert fires, the alert log row now includes a JSON field recording which monitors triggered it, the health score at the time, and the threshold/count values that caused the spike.
**What the user sees:** Column available in DB; alert history tab shows alert entries but does not yet display the trigger_context field in the UI.
**Safe claim:** WRN Store Monitor records what triggered each alert — the specific monitors, health score, and threshold values — so alert history includes diagnostic context.

---

**Capability:** Plugin lifecycle event logging
**Evidence:** `WRNSM_Logger_Hooks::on_plugin_activated()`, `on_plugin_deactivated()`, `EV_PLUGIN_LIFECYCLE`, Timeline tab
**What it actually does:** When any plugin is activated or deactivated in wp-admin, logs a `plugin_lifecycle` row in `wrnsm_events` with plugin file, name, version, and action. Admin-only hook — no frontend overhead.
**What the user sees:** Timeline tab shows "Plugin activated: [name] (version)" or "Plugin deactivated: [name]" entries with ↑/↓ icon in notice color.
**Safe claim:** WRN Store Monitor logs plugin activation and deactivation events so the store history includes environment changes around operational issues.

---

**Capability:** Gateway change event logging
**Evidence:** `WRNSM_Logger_Hooks::on_gateway_option_updated()`, `EV_GATEWAY_CHANGE`, Timeline tab
**What it actually does:** Intercepts WordPress `updated_option` calls matching the `woocommerce_{id}_settings` pattern. Only logs when the `enabled` field changes state. Catches both the WooCommerce Settings page save and the AJAX toggle on the Payments tab. Stores gateway_id and action (enabled/disabled) in the event log.
**What the user sees:** Timeline tab shows "Gateway enabled: stripe" or "Gateway disabled: paypal" entries with ⚙ icon.
**Safe claim:** WRN Store Monitor logs payment gateway enabled and disabled changes so timeline evidence includes gateway state at the time of any failure.

---

**Capability:** Missing data observation log
**Evidence:** `WRNSM_Missing_Data_Log` class, `wrnsm_missing_data_log` table, `WRNSM_Logger_Hooks::on_scan_check_missing_data()`
**What it actually does:** After every scan (priority 30), runs 5 checks for silent absence signals. Writes to `wrnsm_missing_data_log` with 24h deduplication per check_type. 30-day retention pruned by `wrnsm_daily_scan` action.

Check types implemented:
- `no_orders_48h` — no wc_get_orders() results in past 48h → warning
- `no_successful_payment_72h` — no EV_PAYMENT_COMPLETED in event log for 72h → warning
- `gateway_not_configured` — 0 enabled payment gateways → critical
- `no_recent_scan` — auto-scan enabled in settings but no AS action scheduled → notice
- `cron_overdue` — wp_cron monitor result = critical → warning

**What the user sees:** Timeline tab shows "! No orders in past 48h — 0 orders found in past 48h" entries in warning/critical color.
**Safe claim:** WRN Store Monitor records missing diagnostic data observations after every scan.

---

## Partially Implemented Capabilities

**Capability:** Trigger context visible in Alert History tab UI
**What exists:** `trigger_context` column is written to `wrnsm_alert_log` on every critical scan and checkout spike alert.
**What is missing:** The Alert History tab (`tab-alerts.php`) does not yet display the `trigger_context` field — it exists in the database but is not surfaced in the admin UI.
**Safe wording:** Alert trigger context is stored and available for future diagnostic display.
**Unsafe wording:** Do not claim the Alert History tab shows trigger context details — the UI has not been updated.

---

**Capability:** Snapshot data accessible for diagnosis
**What exists:** `snapshot_data` is written to `wrnsm_scan_history` on every scan with full monitor-level evidence.
**What is missing:** No admin UI displays snapshot data. The Diagnostic Export JSON (`export_diagnostic()`) does not yet include `snapshot_data` from scan history rows.
**Safe wording:** Scan evidence is captured and stored.
**Unsafe wording:** Do not claim snapshot data is available for review in any current admin screen.

---

## Missing Capabilities

**AI submission layer** — Sprint 2 or later. Evidence is collected but never sent to an AI service automatically. The existing `WRNSM_AI_Advisor` (which requires a manual Anthropic API key) is separate and scoped to interactive triage, not automated diagnosis from snapshot data.

**WRN Hub integration** — Explicitly deferred. No connection to WRN Hub exists for AI-assisted triage.

**Diagnostic Export including snapshot data** — `WRNSM_Report_Exporter::export_diagnostic()` does not yet include `snapshot_data` from scan history rows. Would be a small Sprint 2 addition.

**Alert History UI — trigger context display** — Column written, not displayed.

**Missing data log admin screen** — Entries appear in Timeline only. No dedicated admin panel, no alerting when thresholds are crossed (e.g. no email when `no_orders_48h` fires).

---

## Requires Configuration

- Plugin lifecycle and gateway change logging: **fully automatic, no configuration required**.
- Missing data checks: **fully automatic, run after every scan, no configuration required**.
- Snapshot storage: **fully automatic, no configuration required**.

---

## Requires Human Service

The entire evidence layer supports WebReadyNow human diagnosis but does not replace it. Specifically:

- When `no_orders_48h` fires, a human must investigate whether this is a store problem (checkout broken, traffic issue) or expected (seasonal, sale ended).
- When `gateway_not_configured` fires, a human must verify and correct the gateway credentials.
- When `cron_overdue` fires, a human or developer must investigate the server WP-Cron configuration.
- When `trigger_context` shows a monitor newly-critical, a human must diagnose whether it is a false positive, a transient condition, or a genuine issue.

---

## Claim Safety Review

| Claim | Classification |
|---|---|
| WRN Store Monitor records diagnostic events inside WooCommerce | Ready to claim |
| WRN Store Monitor logs plugin activation and deactivation events | Ready to claim |
| WRN Store Monitor logs payment gateway enabled/disabled changes | Ready to claim |
| WRN Store Monitor stores scan snapshot evidence | Ready to claim |
| WRN Store Monitor records what triggered alerts | Ready to claim |
| WRN Store Monitor records missing diagnostic data after scans | Ready to claim |
| WRN Store Monitor shows lifecycle, gateway, and missing data in Timeline tab | Ready to claim |
| WRN Store Monitor prepares evidence for future AI-assisted diagnosis | Safe with wording limits — must say "future" and "prepares", not "performs" |
| WRN Store Monitor helps WebReadyNow investigate faster | Safe with wording limits — say "helps" and "WooCommerce-layer evidence", not "diagnoses" |
| WRN Store Monitor diagnoses root cause | Unsafe — not built |
| WRN Store Monitor sends data to AI automatically | Unsafe — not built |
| WRN Store Monitor has WRN Hub integration | Unsafe — not built |
| WRN Store Monitor automatically fixes issues | Unsafe — not built |
| WRN Store Monitor prevents checkout failures | Unsafe — not built |

---

## Product vs. Service Boundary

**Plugin handles independently:**
- Capturing and storing diagnostic events (payment, scan, checkout, lifecycle, gateway)
- Building scan snapshots and missing-data observations
- Recording alert trigger context
- Displaying all evidence in the Timeline tab

**WebReadyNow human service handles:**
- Interpreting missing data signals (is `no_orders_48h` a real problem or expected?)
- Investigating gateway credential issues discovered by `gateway_not_configured`
- Root-cause diagnosis when trigger_context shows a newly-critical monitor
- Fixing identified issues (WP-Cron, plugin conflicts, gateway configuration, server issues)
- Deciding when a diagnostic observation requires action versus monitoring

---

## Development Roadmap Recommendations

**Must build before public claim:**
- License enforcement — `WRNSM_License::is_valid()` is never called; all features run free. Hard blocker before selling.

**Should build soon (Sprint 2 candidates):**
- Include `snapshot_data` in Diagnostic Export JSON
- Display `trigger_context` in Alert History tab UI
- Diagnostic Payload Builder — structured data structure ready for future AI submission
- Missing data alerting (email/Slack when `no_orders_48h` or `gateway_not_configured` fires)

**Nice to have:**
- Missing data log dedicated admin panel
- Snapshot comparison (diff between two scan snapshots)
- WRN Proxy API (replace direct Anthropic key with managed proxy)
- WRN Hub docs seeder rewrite for v1.14.0 features

**Not needed now:**
- AI submission layer — Sprint 2+ after Sprint 2 diagnostic payload is validated
- WRN Hub AI integration — later sprint, requires API design

---

## Recommended Next Agent

No next agent needed yet.

Immediate action: Run manual tests against the v1.14.0 code before Sprint 2.
- Verify `snapshot_data` is written after a scan
- Verify `trigger_context` is written after a critical alert
- Verify `plugin_lifecycle` and `gateway_change` events appear in the Timeline
- Verify `wrnsm_missing_data_log` receives entries after a scan on a test site with low order activity
- Verify 24h deduplication prevents duplicate `check_type` entries

After tests pass: Plan Sprint 2 (Diagnostic Payload Builder) as a small, focused sprint before attempting AI submission.

---

## Knowledge Base Update Recommendation

**Update level:** Level 1 (minor factual update — new verified capabilities, no strategic change)

**Files updated in this session:**
- `06-products/wrn-store-monitor.md` — version 1.14.0, Sprint 1 roadmap entry, new capability sections, updated "What It Does NOT Do"
- `03-safe-claims.md` — Sprint 1 safe claims added under WRN Store Monitor section
- `09-agent-outputs/codebase-audits/wrn-store-monitor-sprint-1-evidence-layer-v1.14.0.md` — this file
- `09-agent-outputs/product-alignment/wrn-store-monitor-ai-diagnostic-system.md` — strategic alignment record

**Decision record needed:** No — Sprint 1 was pre-planned. A decision record would be appropriate if the AI submission strategy is confirmed in Sprint 2 planning.

**Output folder:** `09-agent-outputs/codebase-audits/`
