# Codebase Capability Audit: WRN Pricing Rules Pro (v1.2.1) — WRN Hub Integration

**Audit date:** 2026-06-29
**Agent:** Codebase Capability Auditor Agent
**Plugin audited:** `wr-price-list-pro` v1.2.1 (samuraifireworks.com)
**Server audited:** WRN Hub v1.1.0 (webreadynow.com)
**Status:** Post-fix — launch blockers resolved

---

## Audit Scope

Files reviewed in `wr-price-list-pro`:

| File | Purpose |
|---|---|
| `wr-price-list-pro.php` | Bootstrap, constants |
| `includes/class-wrlp-plugin.php` | Init — wires all classes, update client, cron |
| `includes/class-wrlp-license.php` | WRN Hub REST calls: activate, deactivate, validate; cache; grace period; cron; test mode |
| `includes/class-wrn-update-client.php` | Update checker — polls WRN Hub, injects into WP update transient |
| `includes/class-wrlp-features.php` | Feature gate wrapping `is_valid()` into free-plugin filter hooks |
| `includes/class-wrlp-admin.php` | Admin UI — license AJAX handlers, feature section guards, license notice |
| `templates/admin/license-page.php` | License page — active/inactive/expired/suspended/revoked states |

WRN Hub endpoints called:

| Endpoint | Method | Auth |
|---|---|---|
| `wrn-hub/v1/license/activate` | POST | Key in body |
| `wrn-hub/v1/license/deactivate` | POST | Key in body |
| `wrn-hub/v1/license/validate` | POST | Key in body |
| `wrn-hub/v1/update/{slug}` | GET | None (public) |
| `wrn-hub/v1/download/{slug}?key={license}` | GET | Key in query string |

---

## Executive Capability Decision

The WRN Hub ↔ WRN Pricing Rules Pro integration is architecturally correct and production-ready. Activation, validation, caching, grace period, update injection, and download flow all hit the right endpoints with the right request shapes. Three launch blockers were identified in the initial audit and have been resolved by code changes on 2026-06-29. The plugin is ready for the Marketing & Documentation Agent to write customer-facing copy about licensing.

---

## Implemented Capabilities

| Capability | Evidence | Safe claim |
|---|---|---|
| License activation via WRN Hub | `class-wrlp-license.php:94–124` → POST `/activate` | Entering a valid license key activates Pro features immediately. |
| License validation with 24h cache and grace period | `class-wrlp-license.php:46–72, 191–231` | License validity is verified daily. A 7-day grace period applies when the server is unreachable. |
| Daily re-validation via WP-Cron | `class-wrlp-license.php:150–160`; plugin activation hook | License validity is re-checked automatically each day. |
| License deactivation | `class-wrlp-license.php:130–143` | Deactivating the license clears the site record locally and frees the slot when the server is reachable. |
| Test mode and staging override | `class-wrlp-license.php:74–76, 166–185`; `WRLP_API_BASE` constant | Internal only — not a customer-facing claim. |
| Pro feature gating at filter and admin UI layer | `class-wrlp-features.php:30–43`; `class-wrlp-admin.php:64–69, 86–88, 172–174, 272–274` | Pro features activate with a valid license and stop when the license is confirmed invalid. |
| Update detection via WRN Hub update endpoint | `class-wrn-update-client.php:50–85` | Updates appear in the WordPress plugin updates screen. |
| Authenticated plugin download | `class-wrn-update-client.php:61–64` | Updates install through a license-authenticated endpoint. |
| Plugin info popup (changelog) | `class-wrn-update-client.php:90–113` | Changelog is visible from the standard "View details" link. |
| License admin page UI | `templates/admin/license-page.php` | Plan, renewal date, activation count, and key are shown when active. |

---

## Fixed Launch Blockers (resolved 2026-06-29)

### Fix 1 — False email delivery claim removed

**Problem:** License page said "Enter the license key from your purchase confirmation email." WRN Hub does not email license keys.

**Fix:** Changed to "Enter the license key from your WebReadyNow account at webreadynow.com/my-account."

**File changed:** `templates/admin/license-page.php:93`

---

### Fix 2 — Suspended and revoked license states now handled

**Problem:** WRN Hub can return `suspended` or `revoked` error codes. The Pro plugin was storing both as generic `invalid` status and showing no explanation — the customer saw a blank key-entry form. During activation, suspended/revoked keys produced "Something went wrong."

**Fixes applied:**

- `class-wrlp-license.php:222–224` — `validate_with_server()` now preserves `suspended` and `revoked` as distinct stored status values alongside `expired`. Any other error falls through to `invalid`.
- `class-wrlp-license.php:271–280` — `error_message()` now includes specific messages for `suspended` and `revoked` used during the activation AJAX flow.
- `templates/admin/license-page.php:83–102` — License page inactive section now shows distinct notices for `expired`, `suspended`, and `revoked` statuses, each with a link to webreadynow.com/my-account.

**Suspended message:** "Your subscription is currently on hold. Please update your payment method or manage your subscription in your WebReadyNow account."

**Revoked message:** "This license is no longer active. Please contact WebReadyNow support or manage your account."

---

### Fix 3 — Admin notice added when license lapses

**Problem:** After the 7-day grace period, Pro features went silent. Tabs and settings sections disappeared with no explanation unless the store owner happened to visit the Pro License page.

**Fix:** Added `WRLP_Admin::render_license_notice()` hooked to `admin_notices` via `register_hooks()`. The notice fires on `wr-price-list-for-woocommerce` and `wrlp-license` admin pages when:
- A license key has been stored (`OPT_KEY` is non-empty)
- The stored status is `invalid`, `expired`, `suspended`, or `revoked`
- The current user has `manage_woocommerce` capability

The notice does not fire during the grace period because `OPT_STATUS` stays `active` while the server is unreachable and has not returned a confirmed failure.

**Files changed:** `includes/class-wrlp-admin.php` — `register_hooks()` and new `render_license_notice()` method.

---

## Remaining Known Limitations

These were identified in the audit but are not launch blockers. Tracked for future sprints.

| Issue | Impact | Priority |
|---|---|---|
| Deactivation is fire-and-forget — API result not checked | Ghost activations accumulate silently if deactivation API call fails | Should fix soon |
| No documentation link in plugin UI | Customers have no in-plugin path to docs at webreadynow.com/docs | Should fix soon |
| Plan label mismatch — `developer` stored by WRN Hub may not match `business` key in Pro plugin `$plan_labels` | 5-site plan may render as "Developer" via `ucfirst()` fallback | Should fix soon |
| No license-expiry proactive email | License lapse is only visible reactively via admin notice or license page | Nice to have |

---

## Safe Claims

Authorized after this audit and post-fix verification:

- WRN Pricing Rules Pro connects to the WebReadyNow license server for activation and ongoing validation.
- Entering a valid license key activates Pro features on the site immediately.
- License validity is automatically re-checked daily when WP-Cron is functioning.
- A 7-day grace period keeps Pro features active if the WebReadyNow license server is temporarily unreachable.
- Pro features are gated behind license validation and stop when the license is confirmed invalid.
- Updates appear in the standard WordPress plugin updates screen.
- Plugin updates are downloaded through a license-authenticated WRN Hub endpoint.
- Customers can manage their subscription and license from their WebReadyNow account at webreadynow.com/my-account.
- Expired, suspended, revoked, and invalid license states each show specific customer-facing guidance.

**With wording limits:**
- Deactivating a license frees the site slot — *add "when the WebReadyNow license server is reachable."*
- One-click updates — *add "when the license is active and webreadynow.com is reachable."*

---

## Unsafe Claims

Do not use:

- License keys are emailed automatically after purchase.
- Deactivation always frees the site slot.
- License status messages cover every possible server or billing situation.
- Updates work without a stored license key.
- The plugin can update if outbound HTTPS requests to webreadynow.com are blocked.
- Pro features remain available indefinitely without a valid license.

---

## Recommended Next Action

**Marketing & Documentation Agent** — The integration is clean and launch blockers are resolved. Safe claims above are ready to use as inputs for:
- "How licensing works" section of the product page
- Pro License admin page copy (already updated by Fix 1)
- FAQ entries about license management, site moves, and renewal

The Pro feature capabilities themselves (discount rules, quantity tiers, customer lists, etc.) still require their own Codebase Capability Audit before feature-specific product copy can be written.
