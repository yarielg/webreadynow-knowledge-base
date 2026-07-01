# Product & Offer Alignment: WRN Store Monitor — AI Diagnostic System

**Date:** 2026-06-29
**Status:** Sprint 1 complete. Sprint 2 not yet started.
**Agent:** Product & Offer Alignment Agent (recorded by Codebase Capability Auditor)

---

## Strategic Decision

WRN Store Monitor is the evidence layer for WooCommerce diagnosis.

The plugin's role in the AI Diagnostic System is:
1. Capture what happened (order events, payment events, checkout errors, scan results)
2. Preserve evidence at scan time (snapshot data, environment fingerprint)
3. Record what is missing (silent absence signals via `wrnsm_missing_data_log`)
4. Record what triggered each alert (trigger context in alert log)
5. Support future AI-assisted triage with structured, locally-stored evidence

The plugin does NOT:
- Diagnose root cause automatically
- Submit data to AI automatically
- Connect to WRN Hub for AI triage
- Fix identified issues
- Replace human WooCommerce diagnosis

WebReadyNow managed service fills the gap between detection and resolution.

---

## Sprint Architecture

### Sprint 1 — Diagnostic Evidence Layer (v1.14.0) — COMPLETE

Goal: Enrich existing evidence so future AI diagnosis has better data to work with.

What was built:
- `wrnsm_scan_history.snapshot_data` — compact per-monitor evidence + environment fingerprint per scan
- `wrnsm_alert_log.trigger_context` — what caused each alert, not just delivery status
- `wrnsm_missing_data_log` — new table for silent absence signals (5 check types, 24h dedup, 30-day retention)
- `plugin_lifecycle` event type — activated/deactivated with name and version
- `gateway_change` event type — enabled/disabled state changes with deduplication
- Timeline tab updated to show all new entry types

What was deliberately NOT built:
- AI submission layer — data stays in the WordPress database only
- WRN Hub connection — explicitly deferred
- Marketing copy — deferred pending test completion and claim verification

### Sprint 2 — Diagnostic Payload Builder (PLANNED)

Goal: Build a structured, ready-to-transmit diagnostic payload from accumulated evidence.

Candidates for Sprint 2:
- Include `snapshot_data` in Diagnostic Export JSON
- Display `trigger_context` in Alert History tab UI
- Define a standard diagnostic payload schema (what gets sent, in what format)
- Missing data alerting (email/Slack when critical check types fire)
- Snapshot comparison utility

Do NOT include in Sprint 2:
- Actual AI submission (Sprint 3+)
- WRN Hub integration (Sprint 3+)
- License enforcement (handle separately as a pre-release blocker)

### Sprint 3+ — AI Submission Layer (DEFERRED)

Only after Sprint 2 payload is validated and WRN Proxy API is designed.

---

## Product vs. Service Boundary

| Layer | Handled by |
|---|---|
| Detection of WooCommerce operational signals | Plugin |
| Evidence capture and storage | Plugin |
| Missing data observation | Plugin |
| Alert delivery and trigger recording | Plugin |
| Timeline display of all evidence | Plugin |
| AI-assisted triage (future) | Plugin (Sprint 3+) via WRN Proxy API |
| Root-cause investigation | WebReadyNow managed service |
| Gateway dashboard verification | WebReadyNow managed service |
| Fix implementation | WebReadyNow managed service or developer |
| Webhook repair, plugin conflict isolation | WebReadyNow managed service |
| Checkout testing after fix | WebReadyNow managed service |

---

## Offer Model Implications

The evidence layer strengthens the Managed Monitoring offer:

- "WebReadyNow reviews WRN Store Monitor evidence and diagnoses issues" is now more defensible because the plugin preserves structured scan-time evidence, alert trigger context, and missing-data signals.
- The plugin-only tier gains value: self-serve users get a richer Timeline and more structured evidence to share with developers.
- The Plugin + Setup tier remains valuable: configuring auto-scan, alert thresholds, and gateway monitoring still requires setup.

No new tier or pricing change is implied by Sprint 1 alone. Sprint 2 (payload builder) and Sprint 3 (AI submission) will re-open this question.

---

## Safe Claim Boundary — AI Diagnostic System

**Safe after Sprint 1:**
- WRN Store Monitor captures structured WooCommerce-layer evidence.
- WRN Store Monitor preserves diagnostic history for investigation.
- WRN Store Monitor records missing signals that may indicate silent failures.
- WRN Store Monitor helps prepare evidence that supports faster investigation.

**Safe with limits after Sprint 1:**
- "Prepares evidence for future AI-assisted diagnosis" — only with "future" and "prepares", never "performs" or "sends".

**Unsafe until Sprint 3:**
- AI diagnosis is performed automatically.
- Evidence is sent to AI for triage.
- WRN Hub AI integration exists.
- Root cause is identified without human review.

---

## Pre-Release Blockers (unchanged)

1. License enforcement — hard blocker before selling
2. WRN Hub docs seeder rewrite — must reflect v1.14.0
3. WRN Proxy API — replace direct Anthropic key
4. Landing page copy — cannot be written until license enforcement is complete and claim audit is done

---

## Recommended Next Step

1. Run manual tests for v1.14.0 (see codebase audit for test list).
2. After tests pass, plan Sprint 2 scope (Diagnostic Payload Builder).
3. Do not start Sprint 3 (AI submission) until Sprint 2 payload schema is validated against real scan data.
4. Do not write landing page copy until license enforcement is implemented and a full claim audit is run.
