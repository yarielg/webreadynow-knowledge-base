# Codebase Capability Audit: WRN Store Monitor — Diagnostic & AI Features

**Date:** 2026-06-30
**Agent:** Codebase Capability Auditor
**Plugin version audited:** v1.16.0 (codebase state at audit time)
**Scope:** Diagnostic payload, incident context, AI diagnosis, and all data sources that feed those features

---

## Audit Scope

Files inspected:

- `includes/class-wrnsm-ai-diagnosis.php` — AI call, prompt construction, response parsing, storage, rendering
- `includes/class-wrnsm-diagnostic-payload.php` — payload assembly, all section builders, scrubbing, quality scoring
- `includes/class-wrnsm-diagnostic-payload-store.php` — payload table, save/read, pruning
- `includes/class-wrnsm-incident-context.php` — incident form fields, save/get, single-record pattern
- `includes/class-wrnsm-admin.php` — `handle_run_full_diagnosis()`, `render_diagnosis_details()`, AJAX flow
- `includes/class-wrnsm-settings.php` — AI-related settings (api_key, ai_diagnosis_enabled)
- `includes/class-wrnsm-event-logger.php` — event types, get_stats(), available aggregate queries
- `admin/views/tabs/tab-diagnosis.php` — wizard UI, user-facing form fields and flow
- KB: `03-safe-claims.md`, `06-products/wrn-store-monitor.md`, `09-agent-outputs/product-alignment/wrn-store-monitor-ai-diagnostic-system.md`

Not inspected (out of scope for this audit):
- Checkout probe classes, payment health class, AI advisor per-monitor conversation classes

---

## Executive Capability Decision

The diagnostic system is **structurally complete but analytically weak**. The payload assembly is solid — it collects from 8 data sources, applies privacy scrubbing, and stores results properly. The prompt schema is well-designed with honest limitation declarations. However, two critical quality problems undermine the output: (1) the system runs on **claude-haiku-4-5**, the lowest-capability Claude model, using it for nuanced pattern diagnosis where it will consistently produce generic responses; (2) the user message is capped at **6,000 characters**, which is insufficient to transmit full evidence and causes silent truncation of plugin lists and event history. Additionally, several high-value data sources that already exist in the plugin — payment health gateway stats, checkout probe state, checkout error rates, customer risk aggregates — are **not included in the payload**, leaving significant diagnostic signal on the table. The system can safely be described as "AI-assisted pattern triage based on available store evidence" but cannot be described as producing reliable, actionable diagnosis for payment or checkout issues until the model, token budget, and payload completeness are improved.

---

## Implemented Capabilities

**1. Diagnostic Payload Assembly**
- **Evidence:** `WRNSM_Diagnostic_Payload::build()` — `class-wrnsm-diagnostic-payload.php:84`
- **What it actually does:** Assembles a 13-section payload from scan history, event log, alert log, missing data log, incident context, order status counts, active plugins, and environment. Applies recursive scrubbing for secrets and PII. Computes payload hash.
- **What the user sees:** A "technical details" collapsed section in the Diagnosis tab with payload quality score, flagged monitors, health trend, raw JSON, and download button.
- **Safe claim:** WRN Store Monitor assembles a structured diagnostic snapshot from scan results, event history, alert records, and missing data observations.

**2. Incident Context Form**
- **Evidence:** `WRNSM_Incident_Context::save()` — `class-wrnsm-incident-context.php:27`
- **What it actually does:** Stores 8 admin-entered fields: `symptom_category`, `problem_description`, `started_at`, `is_ongoing`, `is_intermittent`, `affected_gateway`, `customer_visible_error`, `recent_changes`. Single-record in wp_options — each save overwrites.
- **What the user sees:** Wizard step 1: pill radio for symptom category, 3 core text fields, optional advanced fields for gateway and error message.
- **Safe claim:** WRN Store Monitor captures structured incident context (symptom type, description, start date, affected gateway, customer-visible error) to improve diagnostic payload completeness.

**3. AI Diagnosis Generation**
- **Evidence:** `WRNSM_AI_Diagnosis::generate()` — `class-wrnsm-ai-diagnosis.php:95`
- **What it actually does:** Takes a saved payload row, builds system + user messages, calls Anthropic API (`claude-haiku-4-5-20251001`, 4096 max_tokens), parses structured JSON response, saves to `wrnsm_ai_diagnoses` table, returns rendered HTML.
- **What the user sees:** Diagnosis result with confidence badge, classification, plain-English summary, safe-to-conclude list, suggested-only list, cannot-determine list, recommended next steps (3-column), missing data impact, technical summary, evidence used, AI limitations note.
- **Safe claim:** WRN Store Monitor generates an AI-assisted diagnostic draft based on available store evidence when an Anthropic API key is configured. The draft identifies patterns and suggests next steps; it does not confirm root cause.

**4. AI Prompt System**
- **Evidence:** `system_prompt()` and `build_user_message()` — `class-wrnsm-ai-diagnosis.php:159–368`
- **What it actually does:** System prompt defines 13 required output fields, prohibits false-certainty claims, declares structural data absence (gateway logs, PHP logs, server logs, debug.log, order notes, device info). User message is a plain-text formatted dump of all payload sections, capped at 6,000 characters. Prefill `{` forces JSON-only output.
- **Safe claim:** The prompt is honest about evidence gaps and instructs the AI not to speculate beyond available data.

**5. Payload Storage and History**
- **Evidence:** `WRNSM_Diagnostic_Payload_Store` — `class-wrnsm-diagnostic-payload-store.php`, `WRNSM_AI_Diagnosis::save()` — `class-wrnsm-ai-diagnosis.php:470`
- **What it actually does:** `wrnsm_diagnostic_payloads` table holds up to 10 payloads (FIFO prune). `wrnsm_ai_diagnoses` table holds up to 50 diagnosis results. Latest payload and latest diagnosis per payload are retrievable. Diagnosis linked to payload by `payload_id`.
- **Safe claim:** WRN Store Monitor retains the last 10 diagnostic payloads and the last 50 AI diagnosis results for review.

**6. Privacy Scrubbing**
- **Evidence:** `WRNSM_Diagnostic_Payload::scrub_array()` — `class-wrnsm-diagnostic-payload.php:635`
- **What it actually does:** Recursively removes keys matching `key|secret|password|token|webhook|nonce|salt|auth|pass|api|private|credential` pattern. Strips Stripe key patterns and 16-digit card-like strings from values. Caps string values at 400 chars. Depth limit 5 levels.
- **Safe claim:** Diagnostic payloads are scrubbed of API keys, secrets, tokens, and card-like patterns before storage or AI transmission.

**7. Single AJAX Flow (Wizard)**
- **Evidence:** `handle_run_full_diagnosis()` — `class-wrnsm-admin.php:517`
- **What it actually does:** One AJAX call saves incident context → builds payload → saves payload → optionally calls AI diagnosis → returns `{diagnosis_html, details_html, generated_at, payload_id, no_ai}`.
- **Safe claim:** The Diagnosis tab submits one form, executes all diagnostic steps in sequence, and returns the result without page reload.

**8. Rate Limiting**
- **Evidence:** `check_rate_limit()` / `mark_rate_limit()` — `class-wrnsm-ai-diagnosis.php:552–558`
- **What it actually does:** 5-minute per-user transient prevents repeated AI calls.
- **Safe claim:** AI diagnosis generation is rate-limited per user to prevent excessive API calls.

---

## Partially Implemented Capabilities

**1. Payload Quality / Confidence Readiness**
- **What exists:** `build_payload_quality()` returns `confidence_readiness` as `good`, `limited`, or `insufficient`. Logic checks for recent scan, recent events, incident context, gateway context.
- **What is missing:** `good` confidence is practically impossible for payment/checkout symptoms because those categories always degrade to `limited` when gateway logs are absent (which is always). The confidence score doesn't reflect that several key data sources (probe state, payment health stats, error rates) are available but simply not being included in the payload. The system underreports its own capabilities.
- **Safe wording:** "Payload quality assessment indicates whether the AI has sufficient evidence to produce a meaningful diagnostic draft."
- **Unsafe wording:** "Confidence score reflects what the AI knows." (It reflects what the payload builder chose to include, not the full available evidence.)

**2. Event Data in Payload**
- **What exists:** Up to 47 non-scan events + 3 scan events included, prioritized by type (payment_failed first, then checkout_error, gateway_change, etc.).
- **What is missing:** No pre-computed aggregates — the AI receives raw event rows (type, gateway, old_status, new_status, created_at) but not summaries like "3 payment failures in last 24h from Stripe" or "checkout error spike: 15 errors between 14:00–14:30." The AI must infer these patterns from row-level data, which it does unreliably with Haiku.
- **Safe wording:** "Recent events are included in the diagnostic payload."
- **Unsafe wording:** "The AI analyzes payment failure patterns." (It sees raw rows; pattern detection quality depends on model capability.)

**3. Diagnosis Rendering**
- **What exists:** Full structured HTML render: confidence badge, classification, summary, safe conclusions, suggested factors, cannot-determine list, 3-column next steps, missing data impact, technical summary, evidence used, AI limitations note.
- **What is missing:** No diff between this diagnosis and the previous one. No "has this issue been seen before" comparison. No ability to annotate or dismiss sections. No link from diagnosis to the specific monitor tabs.
- **Safe wording:** "AI diagnosis results are displayed with structured confidence levels, suggested actions, and explicit statements of what cannot be determined."

---

## UI-Only / Stub / Demo Capabilities

**None found.** All UI elements in the Diagnosis tab are backed by real logic. The wizard form, loading steps, results panel, technical details section, and re-run button all function.

---

## Missing Capabilities

**1. Payment Health Data in Payload**
`WRNSM_Payment_Health::get_gateway_stats()` and `get_summary()` are never called in `WRNSM_Diagnostic_Payload::build()`. The gateway-level success rates, failure counts, volume, and daily trend that the plugin already calculates and displays in the Reports tab are completely absent from the diagnostic payload. This is the single highest-value missing data source.

**2. Checkout Probe State in Payload**
Current checkout probe status (up/down/degraded, last checked timestamp, consecutive failure count) is not included. This is critical for checkout-category incidents — the AI cannot know if the probe is currently failing.

**3. Pre-computed Aggregate Statistics**
`WRNSM_Event_Logger::get_stats()` returns `payment_failed_24h`, `risk_scored_7d`, `scans_7d` — none are included in the payload. The AI sees individual event rows but not the summary counts that are more reliably interpretable.

**4. Checkout Error Rate and Most Common Error Messages**
The event logger captures checkout error text. A summary of the top 3–5 most frequent error messages in the last 7 days does not exist and is not included in the payload.

**5. Customer Risk Aggregate**
Risk-scored events are in the event log but not aggregated. The payload does not include "N high-risk orders in last 7 days" or the distribution of risk levels.

**6. Incident Context History**
Each incident context save overwrites the previous one (single wp_options record). There is no incident history. If the user runs multiple diagnoses over time, earlier context is lost and cannot be compared.

**7. Automatic Payload Trigger**
`trigger_type` in `wrnsm_diagnostic_payloads` has reserved values `alert` and `scan` for Sprint 3+, but these are not implemented. Payloads are generated manually only — there is no automatic trigger when an alert fires or when health drops to critical.

**8. WRN Hub / Proxy API Integration**
No WRN Hub connection, no managed proxy, no usage caps by license tier. The plugin uses the store owner's own Anthropic key directly.

**9. Comparison / Diff Between Diagnoses**
No mechanism to compare two diagnosis results over time. No "previous diagnosis" reference in the render.

---

## Requires Configuration

**1. AI Diagnosis**
- Requires `ai_api_key` to be set in Settings → AI Advisor (must begin with `sk-ant-`)
- Requires `ai_diagnosis_enabled` to be `true` (default: `true`)
- Without both, the AJAX handler returns a payload-only result with a setup notice

**2. Incident Context Quality**
- Confidence readiness degrades to `insufficient` if no incident context has been saved
- User must enter context before running diagnosis for meaningful output

---

## Requires Human Service

**Root cause confirmation:** The AI produces a pattern-based draft. Confirming root cause requires gateway dashboard access, server error log review, and WooCommerce configuration inspection — all require WebReadyNow or developer involvement.

**Fix implementation:** The AI recommends next steps but does not implement them. Plugin conflict isolation, webhook repair, gateway reconfiguration, and checkout testing after a fix are all service-layer work.

**Gateway dashboard verification:** The payload explicitly declares `gateway_dashboard_logs: unavailable`. Payment processor rejection codes, webhook delivery status, and fraud rule states are inaccessible to the plugin and require manual login to the gateway dashboard.

---

## Claim Safety Review

| Claim | Status | Evidence in code | Safe wording |
|---|---|---|---|
| AI-assisted diagnosis | Safe with limits | `WRNSM_AI_Diagnosis::generate()` — implemented | "AI-assisted diagnostic draft based on available store evidence. Root cause requires human verification." |
| Structured diagnostic payload | Ready to claim | `WRNSM_Diagnostic_Payload::build()` — 13 sections | "WRN Store Monitor assembles a structured diagnostic snapshot from scan, event, alert, and missing data evidence." |
| Privacy-safe payload | Ready to claim | `scrub_array()` — keys + values scrubbed recursively | "Diagnostic payloads are scrubbed of secrets, API keys, and card-like patterns before storage or AI transmission." |
| Payload quality scoring | Safe with limits | `build_payload_quality()` — implemented | "Payload quality score indicates evidence completeness; good confidence requires recent scan, recent events, and incident context." |
| Diagnoses root cause | Unsafe | No root cause confirmation logic | Do not use |
| Detects every checkout failure | Unsafe | Not in scope for this system | Do not use |
| Automatic diagnosis on alert/scan | Missing | trigger_type reserved but not implemented | Do not use until Sprint 3 |
| WRN Hub AI integration | Missing | Not built | Do not use |

---

## Product vs. Service Boundary

**Plugin handles:**
- Evidence capture (events, alerts, snapshots, missing data)
- Incident context collection (8 fields, single record)
- Payload assembly and privacy scrubbing
- AI API call and JSON response parsing
- Diagnosis storage and rendering
- Rate limiting

**Requires WebReadyNow service:**
- Root cause confirmation (gateway logs, server logs, PHP error log review)
- Fix implementation
- Webhook inspection and repair
- Post-fix checkout testing
- Interpreting AI draft in context of the store's full history

---

## Development Roadmap Recommendations

### Must build before the diagnostic feature is meaningfully better

1. **Upgrade model from Haiku to Sonnet** (`WRNSM_AI_Advisor::MODEL` = `claude-haiku-4-5-20251001` → `claude-sonnet-4-6`). This is the single highest-impact change. The prompt and schema are good — the model is the bottleneck. Haiku cannot reliably perform nuanced WooCommerce pattern diagnosis. Sonnet produces substantially better structured reasoning, handles conflicting evidence better, and generates more actionable recommendations. Cost difference is ~5x per call but diagnosis is infrequent.

2. **Add payment health stats to payload.** Call `WRNSM_Payment_Health::get_gateway_stats(7)` and `get_summary(7)` in `WRNSM_Diagnostic_Payload::build()`. Add as `payment_health` section. This gives the AI the most valuable missing signal: gateway-level success rate trends. A store with 74% Stripe success rate over 7 days vs. 94% the previous 7 days is a diagnosis in itself.

3. **Add aggregate statistics section to payload.** Call `WRNSM_Event_Logger::get_stats()` and add as `event_summary`. Minimum useful stats: `payment_failed_24h`, `payment_failed_7d`, `checkout_errors_7d`, `risk_scored_7d`, `checkout_error_spike_24h` (boolean). Raw event rows are not the right unit for AI pattern matching — aggregates are.

4. **Increase the user message character cap from 6,000 to at least 12,000.** At 6,000 chars, the payload is regularly truncated — the plugins list and older events are silently cut. The truncation `...` at line 363 of `class-wrnsm-ai-diagnosis.php` is a hard wall that discards evidence without warning the user. Haiku could handle 6,000; Sonnet and the structured payload deserve more budget.

### Should build soon

5. **Add checkout probe state to payload.** Current probe status (up/down/degraded), last check timestamp, consecutive failure count. For checkout-category incidents this is critical evidence — the AI currently has no idea if the probe is actively failing.

6. **Add top checkout error messages to payload.** Query the event log for the most frequent `checkout_error` meta messages in the last 7 days (top 5). This transforms vague "checkout errors logged" into "Error: 'Sorry, your session has expired' appeared 23 times in the last 48 hours."

7. **Add system-level computed signals section.** Pre-compute for the AI: (a) is the payment failure rate above the normal baseline? (b) are checkout errors spiking? (c) are there repeated issue IDs across 3+ consecutive scans? The payload builder (`recent_scan_summary.repeated_issue_ids`) already computes repeated issues — but it's buried in the user message and the AI may not weight it appropriately.

8. **Store incident context history** (last 5 incidents in a dedicated table instead of overwriting wp_options). Critical for identifying recurring problems and showing the AI that the same symptom has been reported 3 times in 30 days.

### Nice to have

9. **Automatic payload generation on critical alert.** Implement `trigger_type = 'alert'` — build and save a payload automatically when a critical scan alert fires, capturing evidence at the moment of the incident rather than relying on the admin to remember to run diagnosis.

10. **Diagnosis diff / comparison UI.** When re-running diagnosis on the same or similar payload, show what changed between the previous and current AI output.

11. **Structured JSON user message.** The current user message is plain text with formatting. Passing a JSON object as the user message (with the system prompt instructing structured parsing) would be more reliable for the model and easier to test.

---

## Diagnostic System — Next Level Opportunities

**Priority 1: Upgrade the model.** Everything else is secondary. `claude-haiku-4-5` is optimized for speed and cost on simple tasks. Diagnostic triage of a WooCommerce operational failure, where you must reason across payment events, scan results, plugin changes, and environmental signals, is not a simple task. Upgrading to `claude-sonnet-4-6` will produce noticeably better `safe_to_conclude` lists, better `suggested_only` hypotheses grounded in actual evidence, and better `recommended_next_steps` that account for the specific gateway and symptom type. The prompt is already good. The model is the weak link.

**Priority 2: Payment health gateway stats.** The plugin has `WRNSM_Payment_Health` already computing 7/14/30-day success rates, failure counts, and daily trends per gateway. This data is displayed in the Reports tab but never included in the AI payload. A `payment_health` section with per-gateway rates, 7-day trend direction, and "worst performing gateway" summary would give the AI the clearest possible signal about whether a payment problem is gateway-specific, recent, and trending worse.

**Priority 3: Event aggregates, not just rows.** The AI receives up to 47 individual event rows. From those rows it must infer that "there were 8 payment failures yesterday, 0 the day before, and 12 today — all from the same gateway." That inference is unreliable with Haiku and fragile even with Sonnet. Pre-computing `payment_failed_24h`, `payment_failed_7d`, `checkout_errors_7d`, `most_affected_gateway`, and `checkout_error_spike_last_24h` into a summary section, and passing those as named fields, gives the AI explicit facts instead of raw evidence to pattern-match.

**Priority 4: Top error messages.** The most actionable single piece of diagnostic data for a checkout failure is often the literal error string the customer saw. "Sorry, your session has expired" points to session handling. "Payment processing failed" is gateway-side. "Address validation failed" is a different class of problem entirely. The plugin captures checkout error text in event meta. Querying the top 3–5 most frequent error strings and including them in the payload would dramatically improve diagnosis specificity.

**Priority 5: Increase token budget.** The 6,000-character cap on the user message is actively discarding evidence. With Sonnet and a richer payload (adding payment health stats, aggregates, probe state), the payload will need 10,000–15,000 characters to transmit meaningfully. Anthropic's API supports long context well. Increase the cap to match the evidence. Also consider whether some sections (active plugins list) should be summarized differently — e.g., "WooCommerce extensions: [list of WC-specific plugins]" vs. dumping all 30+ plugins.

---

## Recommended Next Agent

**Researcher Agent** — Use it to research what makes AI-assisted WooCommerce diagnosis actually work. Specific questions to answer: (1) What data do gateway-level incidents require for confident diagnosis? (2) How do other WooCommerce diagnostic tools or support workflows collect evidence? (3) What signal-to-noise ratio does the AI need to move from "generic suggestions" to "specific next steps"? (4) What prompt engineering patterns work best for structured diagnostic output with Claude Sonnet vs. Haiku?

After that, the **Product & Offer Alignment Agent** should revisit the service tier model — a better diagnostic system opens the door to a stronger Managed Monitoring offer and a defensible "AI-assisted diagnosis" claim.

---

## Knowledge Base Update Recommendation

- **Update level:** Level 2 — facts changed, safe claims boundary moved
- **Files to update:**
  - `06-products/wrn-store-monitor.md` — Update version to 1.16.x, update AI section to note Haiku model limitation, note missing payload data sources (payment health, probe state, event aggregates)
  - `03-safe-claims.md` — Add clarification: "AI-assisted diagnostic draft — quality depends on model (currently Haiku) and payload completeness; not yet at Sonnet-level reasoning quality"
- **Decision record needed:** Yes — model upgrade decision (Haiku → Sonnet) has cost and pre-release blocker implications
- **Output folder:** `09-agent-outputs/codebase-audits/`
