# WRN Store Monitor — Product File

**Purpose:** Load this file when working on WRN Store Monitor code, marketing, documentation, or the Managed Monitoring service. Cross-reference `03-safe-claims.md` before writing any public-facing copy.

---

## Product Identity

**Name:** WRN Store Monitor
**Type:** WooCommerce monitoring and operational intelligence plugin
**Current version:** 1.13.0
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
- Incident Timeline tab — chronological feed of scans, events, and alerts (last 7 days)
- Diagnostic Export — comprehensive JSON download (scan results, event log, alert history, probe states, environment)
- AI Incident Summary — plain-English summary of active critical issues with business impact
- Response Log — timestamped notes (disabled by default, opt-in in Settings)

### AI Advisor
- AI-assisted diagnosis using store data (requires Anthropic API key)
- Follow-up conversation per monitor
- AI tools: read debug.log, read .htaccess, get WC logs, get active plugins, get store settings, get Action Scheduler jobs, get WC sessions, run SELECT-only DB queries
- Session persistence per user per monitor
- AI Incident Summary for critical states

### Admin Tools
- Diagnostic Export (JSON) — for sending to WebReadyNow for remote diagnosis
- Code Lab (custom PHP snippet runner with AI generation)

---

## What It Does NOT Do

- Automatic issue remediation
- Synthetic checkout testing (no simulated cart/payment submission)
- Full checkout journey monitoring
- Payment submission simulation
- Detection of every possible WooCommerce failure
- Replacement of human WooCommerce support

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

---

## Pre-Release Blockers

1. **License enforcement** — `WRNSM_License::is_valid()` is never called; hard blocker before selling
2. **WRN Hub docs seeder** — needs full rewrite to reflect v1.10.0–1.13.0 features
3. **WRN Proxy API** — replace direct Anthropic key with WRN-managed proxy (usage caps per license tier)
4. **Landing page copy** — product page for webreadynow.com

---

## Architecture Notes

- PHP 8+, WordPress 6.0+, WooCommerce 7.0+
- HPOS-compatible (declared)
- DB tables: `wrnsm_events`, `wrnsm_scan_history`, `wrnsm_ai_feedback`, `wrnsm_alert_log`
- All tables auto-install on `plugins_loaded` via `needs_install()` check
- Action Scheduler for daily scans and continuous probe (group: `wrn-store-monitor`)
- Scan results cached 5 min in `wrnsm_scan_results` transient
- Alert snapshot in `wrnsm_alert_severity_snapshot` option (controls newly-critical detection)

---

## Codebase Location

`C:\wamp64\www\samuraifireworks.com\wp-content\plugins\wrn-store-monitor`
