# Decision Record: WRN Store Monitor Free/Pro Commercial Structure

## Date
2026-07-01

## Agent Used
Product & Offer Alignment Agent

## Topic

Commercial packaging of WRN Store Monitor ahead of its first paid release. Replaces the earlier assumption of testing the plugin on existing client stores (conflicts with WebReadyNow's TOS/client boundaries) with a real self-serve Free/Pro structure sold through WRN Hub.

## Decision

1. **Two standalone plugins, not one plugin with a paywall.**
   - **`WRN Store Monitor`** (Free) — distributed on WordPress.org. Role: acquisition + basic WooCommerce detection/alerting. No license key. No AI. Must be genuinely useful forever — not a crippled trial.
   - **`WRN Store Monitor Pro`** — distributed on webreadynow.com. Role: monetization + diagnosis + advanced monitoring. Standalone plugin, not an add-on. Does not require Free to be installed. The current advanced codebase (v1.17.3, all monitors + AI diagnosis + evidence tools + Code Lab) becomes Pro as-is — it is not rebuilt.

2. **Feature split** (full table recorded in `06-products/wrn-store-monitor.md`):
   - Free: failed/stuck order detection, payment failure pattern detection, inventory/stock issues, environment diagnostics, HPOS status, email deliverability check, basic checkout config check (on-scan only), health score dashboard, daily/manual scan, event log (short retention), email alerts.
   - Pro (license-gated): silent gateway failure / duplicate charge risk / missing payment metadata / stuck payment candidate monitors, customer risk scoring, WC Subscriptions monitoring, continuous checkout probe + cart probe + wc-ajax probe + response-time trending, Slack/webhook alerts + Alert History + delivery logging, Incident Timeline, Diagnostic Payload Builder + Export, AI Advisor, AI Diagnosis + Deepen Evidence + Evidence Tools, Code Lab, Response Log, store priority routing + managed-service handoff panel.

3. **First sellable SKU: `WRN Store Monitor Pro — Annual`.**
   - Customer brings their own Anthropic API key (BYOK).
   - Sold through WebReadyNow, licensed/updated through WRN Hub.
   - Launches first because it requires no WRN Proxy API and no AI credit metering — both confirmed not built in the current codebase.

4. **Later plans (blocked, not cancelled):**
   - `WRN Store Monitor Pro — Monthly` — WebReadyNow-managed AI usage allowance. Blocked until WRN Proxy API / AI metering exists.
   - `WRN Store Monitor Pro — Managed` — WebReadyNow-managed AI usage allowance + Managed WooCommerce Monitoring support layer. Blocked until WRN Proxy API / AI metering AND a defined Managed Monitoring SLA both exist.

5. **Free/Pro coexistence behavior:** If Free is active while Pro is active, Pro shows a persistent admin notice with a one-click "Deactivate Free version" action (nonce + `activate_plugins` capability check). Pro must never auto-deactivate Free without explicit user action. Free and Pro must use separate DB table prefixes, option names, and cron hook names so simultaneous activation cannot collide or fatal.

6. **Next Pro engineering version: `v1.17.4 — Pro Packaging and License Gating`.**
   - Gate the Pro-only feature list using `WRNSM_License::is_valid()`.
   - Add a second, independent AI-plan gate distinct from license validity: valid license **plus** (BYOK key present, for Plan B) **or** (WRN AI credits available, for future Plans A/C).
   - Add the Free-plugin-detection notice described in point 5.
   - Prepare Pro to run standalone (no dependency on Free being installed).
   - Explicitly out of scope for v1.17.4: building the Free WordPress.org plugin, building the WRN Proxy API. Do not move to v1.18.0 as part of this work.

7. **v1.18.0 stays scoped to the existing roadmap (Sprint B — Evidence UX)** unless a fresh scoping pass changes it. The Free plugin becomes its own separate repository and version track, starting at its own `v1.0.0`, and does not consume Pro's version numbers.

## Why This Decision Was Made

- Testing the plugin on live client stores without explicit purchase/opt-in conflicts with WebReadyNow's TOS/client boundaries — a real self-serve commercial structure is required instead.
- `WRNSM_License::is_valid()` exists in the codebase but is called nowhere (confirmed by code inspection, not assumption) — license enforcement has to be wired before any paid sale is possible.
- WRN Proxy API / AI credit metering is confirmed not built. Selling a flat-priced plan with "included AI" before usage is metered is an open-ended cost liability WebReadyNow should not take on. BYOK sidesteps this entirely and is buildable now, which is why it is the first sellable plan.
- WordPress.org requires genuinely useful, non-crippled free plugins. Splitting into two real plugins (rather than a single plugin with a license paywall) is the only compliant path to a WP.org listing.
- `02-current-priorities.md` already listed "WRN Store Monitor pricing model" as an open TODO — this record resolves it.

## Sources Reviewed
- `01-core-context.md`, `02-current-priorities.md`, `03-safe-claims.md`, `04-offer-map.md`
- `06-products/wrn-store-monitor.md`, `07-services/service-map.md`
- `agent-operating-rules.md`, `knowledge-base-update-rules.md`, `agents/product-offer-alignment-agent.md`
- Live codebase: `includes/class-wrnsm-license.php`, `wrn-store-monitor.php`, `PROJECT_NOTES.md`, `CAPABILITY_AUDIT.md` (confirmed `WRNSM_License::is_valid()` is implemented but never called anywhere)

## Safe Claims / Constraints

- Do not claim AI credits are "unlimited" or "included" on any plan before WRN Proxy API + a defined usage cap exist.
- Do not describe the Free plugin as a trial, demo, or limited-time version — it must be positioned and built as permanently useful on its own.
- Do not claim Free/Pro conflict prevention is fully automatic — it is a warning notice plus a one-click user-initiated action, not automatic deactivation.
- Full safe-claim language recorded in `03-safe-claims.md` (see Files Updated below).

## Files, Pages, or Products Affected

- WRN Store Monitor product structure (splits into two plugins)
- WRN Hub (will need to license/serve two separate SKUs plus three plan variants under Pro)
- webreadynow.com Pro landing page (not yet written — still a pre-release blocker, now scoped against three plans instead of one flat offer)
- WordPress.org (new listing, not yet created)

## Action Required

- Engineering: implement `v1.17.4 — Pro Packaging and License Gating` per scope above.
- Engineering (separate, later track): scaffold the standalone Free plugin repo.
- Engineering (separate, later track): WRN Proxy API / AI credit metering.
- Business: define Managed Monitoring SLA (blocks `Pro — Managed` plan).
- Business: define actual price points for all three plans (none set or invented here).
- Business/WRN Hub: create license/plan objects for the three Pro SKUs.

## Status
Planned

## Review Date
On completion of v1.17.4, or 2026-08-01, whichever is first.

## Knowledge Base Update Level
Level 3 — Full update

## Files Updated
- `04-offer-map.md`
- `06-products/wrn-store-monitor.md`
- `02-current-priorities.md`
- `03-safe-claims.md`
- `09-agent-outputs/product-alignment/wrn-store-monitor-free-pro-commercial-structure.md` (this record)

## Notes / TODO
- TODO: Actual price points for all three plans.
- TODO: Managed Monitoring SLA definition (blocks `Pro — Managed`).
- TODO: WRN Hub plan/license objects for the three Pro SKUs.
- TODO: Free plugin repo creation and WP.org submission prep (separate version track, own `v1.0.0`).
- TODO: WRN Proxy API / AI credit metering (blocks `Pro — Monthly` and `Pro — Managed`).
- Not yet committed to git — pending explicit approval per instruction.
