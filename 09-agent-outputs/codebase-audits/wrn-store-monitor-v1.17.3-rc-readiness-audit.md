# Codebase Capability Audit: WRN Store Monitor v1.17.3 — Private Beta / RC Readiness
**Date:** 2026-07-01
**Auditor:** WRN Codebase Capability Auditor Agent
**Plugin version at time of audit:** v1.17.3 (commit `abf5463`)

---

## Audit Scope

**Verified directly in this pass (code read in full):**
- `includes/class-wrnsm-license.php` — activation, validation, grace period, cron
- `includes/class-wrnsm-update-client.php` (`WRN_Update_Client`) + its instantiation in `class-wrnsm-plugin.php`
- `includes/class-wrnsm-settings.php` (AI key field)
- `includes/class-wrnsm-ai-client.php`, `class-wrnsm-ai-response-parser.php`, `class-wrnsm-evidence-tools.php`, `class-wrnsm-admin.php` (`handle_gather_evidence`) — from the v1.17.2/v1.17.3 work done this session
- `admin/views/tabs/tab-code_lab.php`, `snippet-card.php`, `tab-diagnosis.php`, `page-dashboard.php`, `tab-overview.php` — release-facing copy review
- Whole-repo grep for `WRNSM_License::`, `WRN_Update_Client`, gating/enforcement patterns

**Relied on prior audit output (not re-read line-by-line this pass):**
- `09-agent-outputs/codebase-audits/wrn-store-monitor-diagnostic-ai-audit-2026-06-30.md` (Diagnosis/AI layer at v1.16.0)
- `06-products/wrn-store-monitor.md` (KB product file, cross-checked against code where claims were load-bearing)

**Not read in this pass (out of scope — no material change since last audit or not decision-relevant here):**
- Individual monitor scan classes (failed orders, stuck orders, payment health, etc.)
- Checkout probe / Action Scheduler continuous-probe classes
- Alert delivery classes (`WRNSM_Alert_Manager`, `WRNSM_Alert_Log`)
- WRN Hub server-side (not locally accessible to audit)

## Executive Capability Decision

The plugin's detection, logging, alerting, and AI-diagnosis layers are implemented and — for the AI layer specifically — were re-verified and hardened this session (v1.17.2 centralized plumbing, v1.17.3 evidence honesty fix). **The single hard blocker for calling this a sellable Private Beta / RC is licensing: `WRNSM_License::is_valid()` is fully implemented (activate/deactivate/validate/grace-period/cron) but is never called anywhere to gate a feature.** Every feature — including AI diagnosis, Code Labs, and all monitors — runs at full capability with zero license key entered. This is safe to ship as a **free, ungated Private Beta to a small trusted group**, but it is **not safe to sell** in this state. Auto-update is implemented and wired (`WRN_Update_Client` bootstraps unconditionally in `class-wrnsm-plugin.php`), contradicting the KB's stale "not implemented" note — but whether the server-side `wp-json/wrn-hub/v1/update/wrn-store-monitor` endpoint actually responds correctly cannot be verified from this codebase.

## Implemented Capabilities

- **Capability:** AI-assisted diagnosis (Diagnosis tab, standard + Deepen Evidence enriched pass)
  - **Evidence:** `class-wrnsm-ai-diagnosis.php`, `class-wrnsm-ai-client.php`, `class-wrnsm-ai-response-parser.php`, `class-wrnsm-evidence-tools.php`
  - **What it actually does:** Builds a sanitized diagnostic payload, calls Claude Sonnet via a single shared HTTP client, parses a 13-key JSON schema with 4-tier salvage, optionally re-runs with live evidence from 9 controlled read-only tools (fault-isolated per tool as of v1.17.1/v1.17.3).
  - **What the user sees:** A structured card — problem classification, contributing factors, evidence used, missing-data impact, recommended next steps as a numbered timeline, and (as of v1.17.3) an honest "X of 9 diagnostic tools" count on the Deepen Evidence pass.
  - **Safe claim:** "WRN Store Monitor uses AI to analyze store evidence and suggest likely causes and next steps; a human should verify before acting."

- **Capability:** Controlled evidence gathering (no free-form AI DB access)
  - **Evidence:** `class-wrnsm-evidence-tools.php` (9 named methods, no arbitrary SQL), `class-wrnsm-ai-tools.php` (`run_db_query()` retired from `get_definitions()`/`execute()` dispatch as of v1.17.2)
  - **What it actually does:** All live evidence the AI can access goes through 9 fixed, read-only, no-PII methods. The prior path letting the AI author free-form SELECT SQL was removed from active use in v1.17.2.
  - **What the user sees:** Nothing directly — internal safety architecture.
  - **Safe claim:** "AI-assisted diagnosis uses controlled, read-only diagnostic tools — never free-form database queries."

- **Capability:** Code Labs snippet generation, labeled Beta/Experimental
  - **Evidence:** `class-wrnsm-ai-snippet-generator.php`, `class-wrnsm-snippet-runner.php`, `tab-code_lab.php`, `snippet-card.php`
  - **What it actually does:** Generates a PHP snippet via AI from monitor context; store owner manually activates/deactivates/deletes; active snippets execute on `plugins_loaded`.
  - **What the user sees:** "Code Lab — Beta" nav label, experimental-feature description, and an explicit review/stage/backup warning above every generated code block.
  - **Safe claim:** "Code Labs is an experimental feature that generates developer-focused troubleshooting snippets; always review, test on staging, and back up before use." Do not claim automatic fixing.

- **Capability:** Auto-update mechanism
  - **Evidence:** `class-wrnsm-update-client.php` (`WRN_Update_Client`), instantiated unconditionally in `class-wrnsm-plugin.php:143-149`
  - **What it actually does:** Hooks `pre_set_site_transient_update_plugins` and `plugins_api` to surface updates through WordPress's native Plugins page, pulling from `webreadynow.com/wp-json/wrn-hub/v1/update/wrn-store-monitor`, 12-hour cache.
  - **What the user sees:** Standard "Update available" notice on the Plugins page, if the WRN Hub server responds correctly.
  - **Safe claim:** "Updates appear in the standard WordPress plugin updates screen." Cannot confirm the server endpoint is live from this codebase alone.

## Partially Implemented Capabilities

- **Capability:** Licensing system
  - **What exists:** Full activate/deactivate/validate/masked-key-display/grace-period(7-day)/test-mode/daily-cron-refresh logic in `WRNSM_License`.
  - **What is missing:** No call to `WRNSM_License::is_valid()` anywhere outside the class itself. No feature, monitor, AI surface, or admin page checks license validity before running. The plugin behaves identically whether a license is active, expired, or never entered.
  - **Safe wording:** "WRN Store Monitor includes a license management system (activation, validation, renewal grace period)."
  - **Unsafe wording:** "Pro features require an active license" / "features stop working when the license expires" — false as currently coded.

## UI-Only / Stub / Demo Capabilities
None found in the areas audited this pass.

## Missing Capabilities
- **License enforcement** — an already-built system with zero call sites gating anything.

## Requires Configuration
- All AI features require the store owner's own Anthropic API key (bring-your-own-key; WRN Proxy API deferred to v1.19.0).
- Auto-updates and license activation require `webreadynow.com`/WRN Hub reachability — not verifiable from the plugin repo.

## Requires Human Service
- Root-cause confirmation and fix implementation after AI diagnosis — AI produces ranked hypotheses only.
- Gateway dashboard / webhook delivery log inspection — outside the plugin's evidence tools by design.

## Claim Safety Review

| Claim | Verdict |
|---|---|
| "AI-assisted diagnosis with evidence-aware explanation" | Ready to claim |
| "Controlled, read-only evidence tools — no free-form AI database access" | Ready to claim |
| "Code Labs — experimental snippet generation, review before use" | Ready to claim |
| "Automatic updates through WordPress" | Safe with wording limits — client-side confirmed, server-side unverified |
| "License required for premium features" / "Pro-gated" | Unsafe — no enforcement exists |
| "Detects every possible WooCommerce failure" | Unsafe — not claimed anywhere audited |
| "Replaces human WooCommerce support" | Unsafe — not claimed anywhere audited |

## Product vs. Service Boundary

**Plugin handles independently:** detection (monitors, per prior audits + KB), logging, alerting, health scoring, AI-assisted triage/diagnosis with evidence, experimental code-snippet generation, auto-update delivery (client-side).

**Requires WebReadyNow human service:** root-cause confirmation, fix implementation, gateway/webhook dashboard investigation, client communication, custom development.

**Currently missing from the boundary entirely:** a licensing wall. There is no technical difference today between a paying customer and anyone with the plugin files.

## Development Roadmap Recommendations

- **Must build before public/paid claim:** License enforcement — gate at minimum the AI features (Diagnosis, Code Labs, AI Advisor) behind `WRNSM_License::is_valid()`.
- **Should build soon:** Confirm WRN Hub update/license server endpoints respond correctly in production (server-side, not a plugin code change); WRN Hub docs seeder rewrite (stale per KB).
- **Nice to have:** Snippet follow-up/context memory in Code Labs (deferred twice — real feature work, not plumbing).
- **Not needed now:** WRN Proxy API (deliberately deferred to v1.19.0); merging AI system prompts into one shared system.

## Recommended Next Agent
Product & Offer Alignment Agent — the audit surfaces a packaging question (what, if anything, should be free vs. license-gated) that must be decided before license enforcement is built.

## Knowledge Base Update Recommendation
- Update level: Level 2
- Files to update: `06-products/wrn-store-monitor.md` (correct auto-update status)
- Decision record needed: Not for this audit alone — see the paired Product & Offer Alignment decision, which does warrant one.
- Output folder: `09-agent-outputs/codebase-audits/`
