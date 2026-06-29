# WebReadyNow — Current Priorities

**Purpose:** Read this file every session to understand where the business is right now. Update it whenever priorities shift. Do not make decisions based on this file alone — cross-reference `04-offer-map.md` and the relevant product or service file.

**Last updated:** 2026-06-29

---

## Active Development

### WRN Store Monitor — v1.11.0 (Sprint 2 complete)

Sprint 1 (v1.10.0) — Checkout Health Upgrade — DONE
- Cart page HTTP probe
- WooCommerce AJAX (wc-ajax) probe
- Order-received endpoint check
- Continuous checkout probe via Action Scheduler

Sprint 2 (v1.11.0) — Alert Reliability Upgrade — DONE
- Alert delivery logging (wrnsm_alert_log DB table)
- Alert History admin tab
- Failed alert notice in dashboard
- Re-scan button on monitor tabs
- Alert diagnostic panel in Settings
- Send Test Alert tool
- Reset Alert Snapshot tool

Sprint 3 — Diagnosis Support Upgrade — NEXT
- Incident timeline
- Diagnostic export
- AI incident summary
- Managed service handoff panel

Sprint 4 — Managed Service Layer — PLANNED
- Client status labels
- Priority severity routing
- Managed response notes
- Escalation workflow

---

## Active Business Priorities

### Knowledge Base (this)
- Centralized Markdown knowledge base being built now at `C:\WebReadyNow\webreadynow-knowledge-base`
- GitHub repo: TODO — create and link
- Goal: every Claude session across every WebReadyNow project can orient from the same source of truth

### Pre-Release Blockers (WRN Store Monitor)
- License enforcement — `WRNSM_License::is_valid()` is never called; all features run without valid license
- WRN Hub docs seeder rewrite — needs update for v1.10.0+ features
- WRN Proxy API — replace direct Anthropic key with WRN-managed proxy
- Landing page content — write webreadynow.com landing page copy for WRN Store Monitor
- Pattern-based severity thresholds — needs documentation and review

---

## Immediate Decisions Needed

| Decision | Status |
|---|---|
| WRN Hub purpose and product definition | TODO |
| WebReadyNow theme — internal only or sellable product? | TODO |
| WRN Store Monitor pricing model | TODO |
| GitHub repo for knowledge base | In progress |
| webreadynow.com service pages — which to publish next | TODO |

---

## What Should Not Change Right Now

- WRN Store Monitor product direction (detection, logging, alerting, AI triage — not auto-fix)
- Safe claims policy (no guarantees, no automatic fixes, no full checkout protection)
- Agency-first positioning
