# Capability Audit: WRN Store Monitor — Diagnostic & AI Features
**Date:** 2026-06-30
**Auditor:** WRN Codebase Capability Auditor Agent (fork)
**Plugin version at time of audit:** v1.16.0 (per git history)

---

## Audit Scope

**Files read in full:**
- `includes/class-wrnsm-ai-diagnosis.php` — structured diagnosis tab AI
- `includes/class-wrnsm-ai-advisor.php` — monitor-level + global + follow-up + order AI
- `includes/class-wrnsm-diagnostic-payload.php` — payload builder
- `includes/class-wrnsm-diagnostic-payload-store.php` — payload storage
- `includes/class-wrnsm-incident-context.php` — incident context form storage
- `includes/class-wrnsm-settings.php` — AI-related settings
- `admin/views/tabs/tab-diagnosis.php` — diagnosis tab UI (via context)

**Knowledge base loaded:**
- `03-safe-claims.md`
- `06-products/wrn-store-monitor.md`
- `09-agent-outputs/codebase-audits/wrn-store-monitor-sprint-1-evidence-layer-v1.14.0.md`

**Not read in this audit (out of scope):**
- Individual monitor scan classes
- Checkout probe classes
- WRN Hub integration (not yet built)

---

## Executive Capability Decision

The diagnostic and AI infrastructure is architecturally sound and more complete than its surface appearance suggests. However, there are two layers of weakness that cause AI output to feel generic: **the model is underpowered** (Haiku is the cheapest/fastest model, not the best reasoner) and **the payload is missing the most actionable evidence** (no specific affected order IDs/details, no gateway configuration state, no checkout probe results, no WC log excerpts). A confirmed bug in `build_payload_quality()` means payment/checkout/webhook incidents can never reach "good" confidence — they are permanently capped at "limited." The AI UX is fragmented across five different surfaces with no shared context or session history between them. Before marketing AI-assisted diagnosis, the model should be upgraded to Sonnet, the payload enriched with affected order evidence, and the confidence scoring bug fixed.

---

## 1. Diagnostic Evidence Layer — Classification

### Event Logger (`WRNSM_Event_Logger`)
**Classification: Implemented**
- Table: `wrnsm_events`
- Event types: `order_status`, `payment_failed`, `payment_completed`, `risk_scored`, `checkout_error`, `checkout_probe`, `scan_completed`, `plugin_lifecycle`, `gateway_change`
- Fields: `event_type`, `order_id`, `gateway`, `old_status`, `new_status`, `amount`, `score`, `risk_level`, `meta` (JSON), `created_at`
- Query methods: `query()`, `count()`, `get_stats()`, `get_distinct_gateways()`
- Daily caps per event type, 30-day retention
- **What reaches the AI:** 47 non-scan events (priority-ordered by type) + 3 scan events. Amounts deliberately excluded. Meta is recursively scrubbed.
- **Gap:** No gateway-specific error codes captured in event meta. Checkout errors capture the error message but not the HTTP response code or payment gateway's rejection reason.

### Scan History (`WRNSM_Scan_History`)
**Classification: Implemented**
- Table: `wrnsm_scan_history`
- Fields: `score`, `severity`, `snapshot_data` (JSON — per-monitor severity/count/summary + environment fingerprint)
- Sparkline data available for 7–30 days
- **What reaches the AI:** Latest scan health score, severity label, `snapshot_data` (per-monitor severity + count + one-line summary). The full monitor `details` array (specific affected orders) is NOT in snapshot_data — only the summary string.
- **Gap:** Monitor-level detail (specific order IDs, specific error messages, specific amounts) is not stored in the snapshot and does not reach the diagnosis payload.

### Alert Log (`WRNSM_Alert_Log`)
**Classification: Implemented**
- Table: `wrnsm_alert_log`
- Fields: `alert_type`, `channel`, `monitor_id`, `severity`, `status`, `error_message`, `subject`, `trigger_context` (JSON), `created_at`
- `trigger_context` stores: `newly_critical` monitor IDs, health score, threshold vs. count
- **What reaches the AI:** Last 20 alerts with type, channel, severity, status, trigger_context (subject excluded)
- **Gap:** Alert history tells what triggered alerts but not what orders were affected at alert time.

### Missing Data Log (`WRNSM_Missing_Data_Log`)
**Classification: Implemented**
- Table: `wrnsm_missing_data_log`
- Check types: `no_orders_48h`, `no_successful_payment_72h`, `gateway_not_configured`, `no_recent_scan`, `cron_overdue`
- 24h dedup, 30-day retention
- **What reaches the AI:** Last 7 days of observations with check_type, severity, expected, observed, and diagnosis_limitation text
- **Gap:** Only 5 check types. Does not log: no checkout page availability, no Action Scheduler health, no SSL status, no SMTP health.

### Diagnostic Payload Store (`WRNSM_Diagnostic_Payload_Store`)
**Classification: Implemented**
- Table: `wrnsm_diagnostic_payloads`
- Schema: id, created_by, trigger_type, scan_history_id, alert_log_id, symptom_category, incident_summary, payload_json (LONGTEXT), missing_data_json, payload_hash, created_at
- Max 10 rows (oldest pruned)
- trigger_type: only 'manual' implemented; 'alert' and 'scan' are reserved/unbuilt
- **Gap:** Payloads are never auto-generated on alerts or scans — always manual only.

### Payload Quality Scoring
**Classification: Partially implemented — contains a confirmed bug**
- Checks: `has_recent_scan` (scan within 48h), `has_recent_events` (any non-scan event exists), `has_incident_context` (form submitted), `has_gateway_context` (active gateway or event has gateway slug)
- Output: `confidence_readiness` → 'good' | 'limited' | 'insufficient'
- **BUG (class-wrnsm-diagnostic-payload.php, `build_payload_quality()`):** The condition `( ! $has_payment_gateway_logs || ! $has_server_logs )` references `$has_payment_gateway_logs` and `$has_server_logs` as local variables, but they are never assigned in this function scope. PHP treats them as `null` (falsy), so `! $has_payment_gateway_logs` is always `true`. This means: any incident with `symptom_category` in `['payment', 'checkout', 'webhooks']` will ALWAYS receive `confidence_readiness = 'limited'`, never 'good', regardless of evidence quality. The intent was to always declare these as unavailable (which is correct) but the implementation uses undefined variable names.

### Recent Scan Summary
**Classification: Implemented**
- Included in payload: window_days (7), total_scans, latest_score, previous_score, trend (improved/worsened/stable), repeated_issue_ids (monitors appearing critical in 3+ of the last 7 scans), first_seen_at
- **Gap:** AI only gets latest vs. previous score. No raw score series. Cannot see a 7-day score trajectory.

### Recent Events Prioritization
**Classification: Implemented**
- Events sorted by priority: payment_failed > checkout_error > gateway_change > plugin_lifecycle > order_status > checkout_probe > payment_completed > risk_scored
- Then by date DESC within each priority bucket
- **Gap:** Recency is secondary to type. A gateway_change 2 hours ago ranks lower than a payment_failed 6 days ago. For active incidents, recency should be primary.

---

## 2. AI Features — Classification

### Monitor-Level AI (`WRNSM_AI_Advisor::diagnose`)
**Classification: Implemented — Requires configuration**
- Model: `claude-haiku-4-5-20251001` (Haiku — fastest/cheapest, lowest reasoning quality)
- Max tokens: 700
- Rate limit: 300s (5 min) per monitor per user — transient-based
- Input: monitor label, severity, summary, analytics (gateway breakdown, age distribution, revenue at risk), sample orders (first 3 + last 3 of details), full active plugin list + theme + WP constants, event log entries (4–6 rows per relevant event type, last 14 days), monitor meta
- Output schema: `{"diagnosis": "2-3 sentences", "steps": ["step 1", ...]}`
- Storage: None — not persisted; result is rendered directly in admin and lost on page reload
- **Gap:** No persistence. Each "Get AI Diagnosis" click starts fresh. No cross-monitor context. 65-word diagnosis cap limits depth.

### Overview / Store Health AI (`WRNSM_AI_Advisor::global_report` + `incident_summary`)
**Classification: Implemented — Requires configuration**
- Global report: all monitor results + full plugin list + environment → prioritized JSON with 2–4 action items, urgency ratings, cross-monitor pattern detection
- Model: Haiku, 2000 max tokens
- Rate limit: 600s (10 min) per user — transient-based
- Cache: `wrnsm_ai_report_{user_id}` transient, 2-hour TTL
- Incident summary: critical-only monitors + last 5 checkout errors + alert failure flag → plain-English 3-section summary (what is happening / business impact / what needs human diagnosis)
- Incident summary cache: 300s (5 min) per user
- Storage: Both cached in transients only — not persisted to DB
- **Gap:** Global report omits "good" monitors entirely. Cannot detect good monitors masking emerging issues. No history of prior reports.

### Diagnosis Tab AI (`WRNSM_AI_Diagnosis::generate`)
**Classification: Implemented — Requires configuration**
- Model: same as `WRNSM_AI_Advisor::MODEL` = `claude-haiku-4-5-20251001`
- Max tokens: 4096
- Rate limit: 300s per user (simpler — not per-monitor)
- Input: 6000-char capped plaintext message assembled from 10 payload sections
- Prefill trick: assistant message starts with `{` to force JSON output structurally
- Output schema: 13 required keys (see below)
- Storage: `wrnsm_ai_diagnoses` table, max 50 rows, linked to payload_id + payload_hash
- Retrieved by: `get_latest_by_payload_id(payload_id)` — fetches most recent diagnosis for a given payload
- **Gap:** 6000-char cap severely truncates rich payloads. A store with many events will have its payload silently cut, with no indication to the user that truncation occurred.

**Required output schema (13 keys):**
```
problem_classification, likely_contributing_factors, evidence_used,
missing_data_impact, safe_to_conclude, suggested_only, cannot_determine,
recommended_next_steps {store_admin, developer_or_agency, hosting_or_gateway_support},
human_investigation_required, escalation_reason, plain_english_summary,
technical_summary, ai_limitations_note
```
This schema is well-designed and safety-conscious. The problem is the model (Haiku) executing it, not the schema itself.

### Follow-up Conversation AI (`WRNSM_AI_Advisor::followup`)
**Classification: Implemented — Requires configuration**
- Model: Haiku, 1000 max tokens per turn
- Tool-use loop: max 5 iterations
- Tools: get_order_details, get_customer, search_orders, get_order_notes, get_subscription_details, search_subscriptions, get_active_plugins, get_woocommerce_logs, get_store_settings, get_wp_option, get_action_scheduler_jobs, get_wc_sessions, read_debug_log, read_htaccess, run_db_query (SELECT only), add_order_note, update_order_status, update_subscription_status
- Session persistence: history array passed from client (not stored server-side between page loads)
- Context: monitor scan data OR global store data OR specific order for order-analysis context
- **This is the most powerful AI surface** — real-time access to actual order data, logs, and DB
- **Gap:** Session history lives in the browser. Refresh = start over. No server-side conversation persistence.

### Order-Level AI (`WRNSM_AI_Advisor::analyze_order`)
**Classification: Implemented — Requires configuration**
- Triggered from order edit page metabox
- Input: full order info, risk score + signals, last 8 order notes, line items
- Output schema: `{status, headline, diagnosis, action, action_url}`
- Model: Haiku, 500 max tokens
- Storage: None — rendered inline, not persisted
- **Gap:** No PII concern addressed here — customer name, email, phone, billing address are included in the prompt. This is intentional (admin-only tool) but should be noted.

### AI Settings
**Classification: Implemented — Requires configuration**
Available settings:
- `ai_api_key` — direct Anthropic API key (stored in wp_options, plaintext)
- `ai_diagnosis_enabled` — boolean, default true
- No model selection — hardcoded to `claude-haiku-4-5-20251001` across all AI surfaces
- No per-feature enable/disable (monitor AI, global report, diagnosis tab are all controlled by one key)
- No rate limit configuration for users
- No token budget configuration
- No WRN Proxy — store owner provides their own key directly

### AI Storage Tables
**Classification: Implemented (partially)**
- `wrnsm_ai_diagnoses` — diagnosis tab results, 50-row cap, linked to payload_id ✓
- `wrnsm_ai_feedback` — referenced in `WRNSM_AI_Feedback::render_footer()` but not audited in this pass
- Monitor-level AI results: **not stored** — no DB table
- Global report: **transient only** — 2-hour cache, lost on expiry
- Incident summary: **transient only** — 5-min cache
- Follow-up conversation: **not stored** — browser-side history only

---

## 3. Diagnosis Tab Flow — Classification

### Incident Context Form
**Classification: Implemented**
- 8 fields: symptom_category (6 options: checkout/payment/orders/webhooks/performance/unknown), problem_description (1000 chars), started_at (date), is_ongoing (bool), is_intermittent (bool), affected_gateway (key slug), customer_visible_error (500 chars), recent_changes (500 chars)
- Stored as: single `wrnsm_incident_context` wp_option — one record, overwrites on save
- **Gap:** Only one incident context at a time. No way to save context for "the problem last Tuesday" vs "the new problem today." No date range for the incident. No affected customer count. No affected order count estimate from the admin.

### Payload Generation
**Classification: Implemented**
- Triggered manually via AJAX (`wrnsm_run_full_diagnosis` handler)
- Steps: save incident context → `WRNSM_Diagnostic_Payload::build()` → `WRNSM_Diagnostic_Payload_Store::save()`
- Build assembles: health, environment (from snapshot_data), monitors (from snapshot_data), scan_snapshot (raw), order_status_summary (direct DB), recent_scan_summary (last 10 scan events), recent_events (150 event pool filtered), recent_alerts (20 rows), missing_data (7 days), active_plugins (cached 5 min), payload_quality
- **Gap:** Payload is built from stored data only. No real-time data fetch at generation time (events, scan, alerts are whatever was already logged).

### Payload Preview / Export
**Classification: Implemented**
- Admin UI shows: payload quality checklist, flagged monitors (non-good only), health score + scan trend, raw JSON textarea with Copy button, download form
- JSON download via admin-post action with nonce
- **Gap:** Preview shows the payload sections but not the full context the AI will actually receive (the 6000-char truncated plaintext message).

### AI Diagnosis Generation
**Classification: Implemented — Requires configuration**
- Triggered as part of the same AJAX call that builds the payload
- Requires: `ai_api_key` setting, `ai_diagnosis_enabled` = true
- If no API key: returns `no_ai: true` — tab shows a "no AI configured" notice
- If API key present: calls Anthropic, parses structured JSON response, stores in `wrnsm_ai_diagnoses`
- Rate limit: 300s per user

### Diagnosis Storage
**Classification: Implemented**
- Table: `wrnsm_ai_diagnoses` — stores full `diagnosis_json` (all 13 fields), `diagnosis_summary` (500-char truncated `plain_english_summary`), `confidence_level`, `human_investigation_required` flag, `model`, `prompt_version`, `payload_hash`, `created_at`
- Max 50 rows, oldest pruned
- **Gap:** Only the most recent diagnosis per payload_id is rendered. No UI to browse diagnosis history.

### Diagnosis Rendering
**Classification: Implemented**
- `WRNSM_AI_Diagnosis::render()` produces: confidence badge, classification label, human-required badge, draft notice, plain English summary, escalation reason, "safe to conclude" list (open), "suggested only" list, "cannot determine" list, recommended next steps (3 columns: store admin / developer / hosting), missing data impact, technical summary, evidence used, AI limitations note, meta footer (model, prompt version, generated_at)
- Well-structured and safety-conscious rendering

---

## 4. Payload & Data Gaps

Data that EXISTS in the plugin but is NOT fed into the diagnosis payload:

| Data Available | Why Missing from Payload | Impact on Diagnosis |
|---|---|---|
| Specific affected order IDs | Only count + summary in snapshot_data | AI cannot reference specific orders; diagnosis is pattern-only |
| Gateway configuration state | Not collected at payload-build time | AI doesn't know if gateway is in test mode, has valid credentials, or has correct webhook URLs |
| Continuous checkout probe results | Not included in payload sections | AI doesn't know if the checkout page is currently up or down |
| Full monitor detail arrays | Only severity/count/summary in snapshot | AI gets "3 stuck orders" not "Order #1234 stuck 5 days, #1235 stuck 3 days" |
| Order amounts / revenue at risk | Deliberately excluded in Sprint 2 | AI cannot assess financial severity |
| WC log file content | Excluded for privacy | PHP exceptions during checkout are invisible |
| Action Scheduler job status | Not in payload | Failed background jobs affecting checkout are not visible |
| SSL certificate days remaining | Not in payload | Upcoming SSL expiry as a cause is not visible |
| SMTP configuration state | Not in payload | Email deliverability issues as a cause are not visible |
| Debug log (even partial) | Excluded for privacy | Fatal errors at checkout time are invisible |
| Recent order notes | Excluded for privacy | Gateway decline messages written to order notes are invisible |
| Customer device/browser info | Never captured | Cannot identify browser-specific or mobile-specific patterns |
| Webhook endpoint URLs | Scrubbed by key pattern (`webhook` matches SCRUB_KEY_PATTERN) | AI cannot see if webhook URLs are misconfigured |
| Scan trend as a raw series | Only latest + previous score | AI gets "worsened" but not "score was 82, 78, 71, 65, 60 over 5 days" |
| Missing data check for checkout probe | Only 5 check types | No signal for "checkout probe has been failing for 6h" |

---

## 5. AI Prompt & Output Analysis

### Model
All AI surfaces use `claude-haiku-4-5-20251001`. This is the fastest and cheapest Claude model — designed for speed, not complex multi-step reasoning. For pattern-based diagnosis of WooCommerce incidents, this is the primary cause of generic output.

- Monitor-level diagnosis: **700 max tokens** — enough for 2-3 sentences + steps
- Global report: **2000 max tokens** — reasonable for 2–4 priorities
- Structured diagnosis (tab): **4096 max tokens** — adequate, but the 6000-char input cap limits what the model can reason about
- Follow-up: **1000 max tokens per turn** — too low for detailed investigation responses

### Monitor-Level Prompt (from `system_prompt()`)
The prompt is direct, specific, and well-crafted. It instructs the model to:
- Reference actual gateway plugin names
- Include exact WooCommerce admin menu paths
- Identify the single most likely root cause
- Never suggest "contact support" as a first step
- Never invent data not present in context

The problem is not the prompt design — it's the model executing it and the brevity constraints (65-word diagnosis cap).

### Structured Diagnosis Prompt (from `system_prompt()` in `WRNSM_AI_Diagnosis`)
The system prompt is thorough and safety-focused. Prohibitions are specific and correct. The JSON schema is well-designed.

**Key limitation:** The user message is built as a plaintext string with a hard 6000-character cap. When truncated:
- No indication is shown to the user that data was cut
- The model receives an incomplete picture
- The most recent events (at the end of the string) are most likely to be cut

### Why Output Feels Generic — Root Causes

1. **Haiku model**: Cannot perform deep WooCommerce-specific reasoning. Sonnet or Opus would produce materially better diagnosis.
2. **No specific order evidence**: The AI sees "3 failed orders in 7 days via stripe" but not "Order #1234 failed June 28 at 2:15pm with error 'Your card was declined'" — the latter enables specific diagnosis.
3. **No gateway configuration context**: Without knowing if Stripe is in test mode, has valid webhook secret, or has a known configuration issue, the AI can only say "check your gateway settings."
4. **6000-char input cap**: Rich payloads with many events are silently truncated.
5. **Confidence bug**: Payment/checkout/webhook incidents are always "limited" confidence — the model is explicitly told evidence is weak even when it isn't.
6. **No WC log access**: The most actionable diagnostic data (PHP errors, gateway responses) is excluded from the payload for privacy reasons, but it means the diagnosis is always pattern-only.
7. **No checkout probe state**: If the checkout probe has been failing for 2 hours, this is the most critical data point — but it's not in the payload.
8. **Payload built from stored data**: The payload is a snapshot of what was already logged, not a live fetch. A fresh scan is not triggered at diagnosis time.

---

## 6. Privacy & Security Review

### What Is Included in AI Payloads
- Site URL (home_url) — included
- PHP/WP/WC versions — included
- Active gateway slugs (not credentials) — included
- Active plugin names + versions — included (no file paths)
- Monitor severity + summary + issue count — included
- Order ID references in events — included (internal reference, not PII)
- Event types + gateway slug + status transitions — included
- Health score + scan trend — included
- Missing data signals — included
- Symptom category, problem description, started_at, affected_gateway, customer_visible_error, recent_changes — included (admin-provided)

### What Is Excluded
- Customer names, emails, addresses, phone numbers — excluded
- API keys, webhook secrets, tokens, nonces, salts — excluded (SCRUB_KEY_PATTERN: `/key|secret|password|token|webhook|nonce|salt|auth|pass|api|private|credential/i`)
- Stripe key patterns (`sk_live_*`, `sk_test_*`, `pk_live_*`, `pk_test_*`) — scrubbed
- 16-digit card-like numbers — scrubbed
- Order amounts — excluded (Sprint 2 decision)
- Alert email subjects — excluded
- Order notes — excluded
- Raw debug.log — excluded

### Privacy Concerns
1. **API key stored plaintext in wp_options** — standard WordPress pattern but worth noting. Should use encrypted storage or WRN Proxy.
2. **Webhook URLs are scrubbed from event meta**, but `affected_gateway` from incident context (the user-entered gateway slug) IS included. This is fine.
3. **Order-level AI (`analyze_order`)** includes customer name, email, phone — this is admin-only and intentional, but PII does go to Anthropic's API.
4. **No user-facing disclosure** before data is sent to AI — the UI shows the payload preview but does not have an explicit "this will be sent to Anthropic" confirmation step.
5. **`ai_api_key` field uses the user's own key** — all API costs are borne by the store owner. No WRN proxy. This is transparent but means no usage cap enforcement.

---

## 7. AI UX Fragmentation

AI surfaces exist in five disconnected locations:

| Surface | Entry Point | AI Call | Model | Persisted? | Shares Context With |
|---|---|---|---|---|---|
| Monitor tabs | "Get AI Diagnosis" button per monitor | `WRNSM_AI_Advisor::diagnose` | Haiku | No | Nothing |
| Monitor tabs | Follow-up chat | `WRNSM_AI_Advisor::followup` | Haiku | Browser only | Monitor diagnosis in same session |
| Overview tab | "AI Priority Report" button | `WRNSM_AI_Advisor::global_report` | Haiku | Transient 2h | Nothing |
| Overview tab | "AI Incident Summary" (critical only) | `WRNSM_AI_Advisor::incident_summary` | Haiku | Transient 5m | Nothing |
| Order edit page | "Analyze with AI" metabox | `WRNSM_AI_Advisor::analyze_order` | Haiku | No | Nothing |
| Diagnosis tab | "Run Diagnosis" wizard | `WRNSM_AI_Diagnosis::generate` | Haiku | DB (50 rows) | Payload (structured) |

**Problems:**
- A monitor-level diagnosis and the Diagnosis tab produce independent, unrelated results on the same issue
- The follow-up conversation on a monitor tab has no awareness of the structured diagnosis
- Refreshing any page resets all non-DB-persisted AI results
- No way to "escalate" a monitor-level diagnosis to a full structured diagnosis
- No AI result linking any surface to another

---

## 8. Feature Classification Table

| Feature | Classification | Notes |
|---|---|---|
| Event logging (9 types) | Implemented | Amounts, PII excluded from AI payload |
| Scan history with snapshot_data | Implemented | Monitor detail arrays not in snapshot |
| Alert log with trigger_context | Implemented | Full trigger context included in payload |
| Missing data log (5 check types) | Implemented | Only 5 checks — not exhaustive |
| Diagnostic payload builder | Implemented | 6000-char cap, amounts excluded |
| Diagnostic payload store | Implemented | 10-row cap, manual-only trigger |
| Payload quality scoring | Partially implemented | Bug: payment/checkout/webhook always "limited" |
| Incident context form | Implemented | Single record, no history |
| Monitor-level AI diagnosis | Implemented — requires config | Haiku model, not persisted |
| Follow-up AI conversation | Implemented — requires config | Most powerful surface; browser-history only |
| Order-level AI analysis | Implemented — requires config | Includes PII (intentional) |
| Global AI priority report | Implemented — requires config | Transient cache only |
| AI incident summary | Implemented — requires config | Critical-only, transient |
| Structured diagnosis (tab) | Implemented — requires config | Best schema, Haiku model, 6000-char cap |
| AI diagnosis storage | Implemented | 50 rows, linked to payload |
| Model selection | Missing | Hardcoded Haiku everywhere |
| Auto payload generation (on alert) | Missing | trigger_type='alert' reserved but unbuilt |
| Auto payload generation (on scan) | Missing | trigger_type='scan' reserved but unbuilt |
| Gateway configuration in payload | Missing | Active gateway slugs only |
| Affected order evidence in payload | Missing | Count + summary only |
| Checkout probe state in payload | Missing | Probe runs separately |
| WRN Proxy API | Missing | Direct Anthropic key only |
| Conversation persistence (server-side) | Missing | Browser-side only for follow-up |
| Cross-surface AI context sharing | Missing | Five disconnected surfaces |
| Diagnosis history browser | Missing | Latest-only retrieval per payload |
| User-facing data disclosure notice | Missing | No explicit "data will be sent to Anthropic" step |
| Payload truncation warning | Missing | 6000-char cap is silent |

---

## 9. Current Limitations Summary

Why the AI diagnostic output is not yet good enough:

1. **Wrong model for the job.** Haiku is optimized for speed and cost, not WooCommerce-specific diagnostic reasoning. Every AI surface uses it. Sonnet would produce materially better diagnoses.

2. **No specific order evidence in the payload.** The AI reasons about patterns ("3 failed orders") but has no specific order details (date, gateway response, error message, amount) — the minimum evidence needed for actionable WooCommerce diagnosis.

3. **No gateway configuration in the payload.** Test mode status, webhook URL presence, last successful transaction time — none of this is included. "Check your gateway settings" is the best the AI can offer without it.

4. **6000-character payload cap is silent.** Rich payloads are truncated without notice. The model reasons on a partial picture without knowing data was cut.

5. **Confidence scoring bug permanently caps payment/checkout/webhook incidents.** Undefined variables in `build_payload_quality()` mean these categories are always "limited" — the AI is always told its evidence is weak for the most common incident types.

6. **Payload is built from stored data, not live data.** No real-time data fetch at diagnosis time. If the checkout probe is currently failing, the payload won't reflect that unless the probe already ran and logged.

7. **No WC log access in structured diagnosis.** The most actionable diagnostic signal — PHP error logs, gateway response logs — is excluded from the payload. The follow-up AI CAN read logs; the diagnosis tab AI cannot.

8. **Checkout probe results not included in payload.** A continuous probe runs via Action Scheduler, but its current state doesn't reach the diagnosis payload.

9. **Five disconnected AI surfaces with no shared context.** Monitor AI, global AI, order AI, incident summary, and diagnosis tab are independent systems. Diagnosing a problem requires using multiple surfaces and mentally connecting the results.

10. **No persistence for four of five AI surfaces.** Monitor diagnoses, global reports, incident summaries, and order analyses are not stored. A page refresh loses all results. Only the structured diagnosis (tab) is persisted to the DB.

---

## 10. Recommended Questions for the Researcher Agent

Questions the Researcher Agent should answer before the next diagnostic feature design sprint:

1. **Model selection:** What are the realistic quality differences between Haiku, Sonnet, and Opus for WooCommerce-specific diagnostic reasoning? What is the cost/quality tradeoff for a 4096-token structured diagnosis? Would prompt caching on the system prompt materially reduce per-diagnosis cost?

2. **Payload richness:** What are the minimum data points that make a WooCommerce payment failure diagnosis actionable? What do experienced WooCommerce developers look at first when diagnosing a checkout failure? How do tools like Statuspage, Datadog, or Sentry handle structured diagnostic context?

3. **Gateway-specific evidence:** What gateway-specific signals can be captured at the WooCommerce layer (webhooks received/missed, IPN logs, API response codes in order meta) without requiring gateway dashboard access? How do Stripe, PayPal, Square, and WooCommerce Payments record rejection codes in WooCommerce order meta/notes?

4. **Checkout probe integration:** How should continuous probe results (probe_down, probe_recovered) feed into the diagnosis payload? Should a failing probe automatically trigger a payload generation? What is the right architecture for "real-time diagnostic context" vs. "batch snapshot context"?

5. **Conversation persistence:** What are the architecture options for server-side AI conversation history in WordPress? What are the privacy and storage implications of persisting follow-up conversations?

6. **Prompt engineering for WooCommerce diagnosis:** What prompt structures produce the most actionable output from Claude Sonnet for "stuck order" vs. "payment failure" vs. "checkout error" incident types? Are there domain-specific prompting patterns that improve diagnosis quality?

7. **Evidence hierarchy:** In a real WooCommerce incident investigation, what is the priority order of evidence? (order notes → gateway logs → WC logs → plugin conflicts → server logs?) How should this hierarchy inform what the payload prioritizes and what the prompt emphasizes?

8. **Auto-trigger architecture:** Should diagnostic payloads be auto-generated on critical alerts? On scan score drops below a threshold? What are the token/cost implications of auto-generation at scale? What would trigger too many false-positive diagnoses?

9. **Cross-surface AI unification:** What would a unified "WooCommerce incident investigation session" look like — one AI surface that has monitor data, order data, log data, and conversation history? Is this a single tab, a dedicated incident workspace, or something else?

10. **WRN Proxy API design:** What rate limits, per-license token caps, and model routing logic should the WRN Proxy implement? Should different license tiers get different models (Haiku vs. Sonnet)?

---

## Knowledge Base Update Recommendation

- **Update level: Level 2** — audit findings affect product claims and development priorities
- **Files to update:**
  - `06-products/wrn-store-monitor.md` — update version to 1.16.0, note Haiku model hardcoding, note payload quality bug, note 5 disconnected AI surfaces, note payload 6000-char cap
  - `03-safe-claims.md` — add: "AI diagnosis uses Claude Haiku; output quality improves significantly with better payload data and a more capable model" — do NOT claim AI diagnosis is comprehensive until model is upgraded and payload is enriched
- **Decision record needed:** Yes — model upgrade decision (Haiku → Sonnet) is a cost/quality trade-off that should be documented
- **Output saved to:** `09-agent-outputs/codebase-audits/wrn-store-monitor-diagnostic-ai-audit-2026-06-30.md`

---

## Recommended Next Agent

**Researcher Agent** — use the 10 questions in Section 10 above as the research brief. The Researcher should answer the model selection, payload richness, gateway evidence, and prompt engineering questions before any new diagnostic feature is designed or built.
