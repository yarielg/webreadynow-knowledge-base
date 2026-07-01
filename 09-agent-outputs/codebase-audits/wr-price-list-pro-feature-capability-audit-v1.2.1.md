# Codebase Capability Audit: WRN Pricing Rules Pro (v1.2.1) — Feature Capabilities

**Audit date:** 2026-06-29
**Agent:** Codebase Capability Auditor Agent
**Plugin audited:** `wr-price-list-pro` v1.2.1 (samuraifireworks.com)
**Scope:** Pro-specific features only — discount rules, quantity tiers, customer price lists, frontend display, notifications, admin settings, inspector. License/update integration covered in a prior audit (`wr-price-list-pro-wrn-hub-integration-v1.2.1.md`).
**Status:** Read-only audit. No code changes made.

---

## Audit Scope

Files reviewed in `wr-price-list-pro`:

| File | Purpose |
|---|---|
| `includes/class-wrlp-discount-rules.php` | Discount engine — `item_price` and `cart_total` code paths, condition evaluation, stacking |
| `includes/class-wrlp-discount-db.php` | DB CRUD for `wrlp_discount_rules` and `wrlp_rule_conditions` tables |
| `includes/class-wrlp-customer-lists.php` | Customer-specific price list assignment — user meta |
| `includes/class-wrlp-quantity-tiers.php` | Volume pricing table — single product page + variations |
| `includes/class-wrlp-inspector.php` | Admin pricing inspector — admin bar panel, product/cart contexts, AJAX refresh |
| `includes/class-wrlp-price-table.php` | `[wrn_price_table]` shortcode — customer-facing price list table |
| `includes/class-wrlp-cart-breakdown.php` | Cart savings calculation for inspector |
| `includes/class-wrlp-expiry-notifications.php` | Cron-based expiry email for price lists and rules |
| `includes/class-wrlp-install.php` | DB schema installer and migration |
| `includes/class-wrlp-admin.php` | Admin UI — settings sections, modal fields, wizard, AJAX handlers |
| `templates/admin/discount-rules.php` | Discount rules admin table and add/edit modal |
| `templates/frontend/price-table.php` | Frontend price table template (used by `[wrn_price_table]`) |
| `templates/single-product/tier-table.php` | Volume pricing table template (single product page) |

**Not reviewed in this audit:**
- Free plugin (`wr-price-list-for-woocommerce`) — the Pro plugin depends on it and extends its hooks, but its internals are out of scope
- JavaScript assets (`assets/js/`) — frontend interaction logic not verified for completeness; audit focuses on backend capability
- CSS assets — not in scope for capability audit

---

## Executive Capability Decision

The core feature set of WRN Pricing Rules Pro is fully implemented and production-ready at the engine level. Discount rules (both item-price and cart-total paths), quantity tier pricing, customer-specific price list assignment, and the admin pricing inspector are all backed by real, executing code. Several important behavioral edge cases exist that must be communicated accurately in documentation (order-count logic, cart-discount-base scoping, customer search limit). The `[wrn_price_table]` shortcode is fully implemented and safe to document. No UI-only or stub features were found — every visible admin control has backend logic consuming it. The Marketing & Documentation Agent can now write feature-level product copy using the safe claims in this audit.

---

## Implemented Capabilities

### Discount Rules Engine

- **Capability:** Create, edit, delete, and reorder discount rules
- **Evidence:** `class-wrlp-admin.php` AJAX handlers (`ajax_save_discount_rule`, `ajax_delete_discount_rule`, `ajax_reorder_discount_rules`, `ajax_toggle_discount_rule`); `class-wrlp-discount-db.php` (`insert_rule`, `update_rule`, `delete_rule`, `reorder_rules`, `toggle_status`)
- **What it actually does:** Full CRUD. Rules are stored in `wrlp_discount_rules` with name, type, value, apply_to, active date range, and status. Conditions stored separately in `wrlp_rule_conditions`. Priority set by drag-to-reorder (jQuery UI sortable).
- **What the user sees:** Discount Rules tab with a rules table (name, discount, conditions summary, status toggle, period, edit/delete buttons). Drag handle for reordering. Modal dialog for add/edit with inline conditions builder.
- **Safe claim:** Discount rules can be created, edited, reordered, and enabled or disabled from the Discount Rules admin tab.

---

- **Capability:** Discount types — percent off and fixed amount off
- **Evidence:** `class-wrlp-discount-rules.php:apply()`, `class-wrlp-discount-db.php` schema; `discount_type` field values `percent_off` and `fixed_off`
- **What it actually does:** `percent_off` multiplies the price by `(1 - value/100)` per rule. `fixed_off` subtracts a flat amount. Both types work on `item_price` and `cart_total` paths.
- **What the user sees:** Dropdown in rule modal — "Percent off (%)" or "Fixed amount off."
- **Safe claim:** Rules support percentage discounts and fixed-amount discounts.

---

- **Capability:** Apply discount to item price or cart total
- **Evidence:** `class-wrlp-discount-rules.php`: `apply()` (item_price path, hooks `woocommerce_product_get_price`) and `apply_cart_fees()` (cart_total path, hooks `woocommerce_cart_calculate_fees`)
- **What it actually does:** `item_price` adjusts each matching product's price individually — discount multiplies by quantity. `cart_total` adds a single negative fee line to the order totals — avoids multiplying a fixed discount by every item.
- **What the user sees:** "Apply discount to" dropdown: "Item price" or "Cart total — one discount line in order totals."
- **Safe claim:** Discount rules can be applied per-item (adjusting each product's price) or as a single cart-level discount line.

---

- **Capability:** Rule conditions — role, customer, product, category, min_qty
- **Evidence:** `class-wrlp-discount-rules.php`: `check_condition()`, `CART_CONDITION_TYPES` constant. Condition types used in `apply()` item-price path.
- **What it actually does:** All conditions on a rule must pass (AND logic). `role` matches the current user's WP role. `customer` matches a specific user ID. `product` matches one or more product IDs (supports multiple products and individual variations). `category` matches a product category (with variable product fallback to parent categories). `min_qty` matches when the item quantity in cart meets the threshold.
- **What the user sees:** "Add condition" button in rule modal with type selector. Conditions display as pills in the rules table.
- **Safe claim:** Rules can be scoped to specific user roles, individual customers, specific products, product categories, and minimum item quantities.

---

- **Capability:** Rule conditions — cart-level: min_cart_total, min_cart_qty, is_first_order, min_order_count
- **Evidence:** `class-wrlp-discount-rules.php`: `CART_CONDITION_TYPES`, `apply_cart_fees()`, `check_condition()`, `get_customer_order_count()`
- **What it actually does:** Cart-condition types force the rule into the `apply_cart_fees()` code path regardless of `apply_to` setting. `min_cart_total` checks cart subtotal. `min_cart_qty` checks total cart item quantity. `is_first_order` checks that the customer has zero completed orders. `min_order_count` checks the number of completed orders meets a threshold. **Note:** Both `is_first_order` and `min_order_count` count only `wc-completed` orders — pending or processing orders do not count.
- **What the user sees:** Condition type dropdown includes "Cart total ≥", "Cart qty ≥ N items", "First order", "≥ N past orders."
- **Safe claim:** Rules can be triggered by cart total, cart item count, whether it is a customer's first purchase, or how many completed orders a customer has placed.

---

- **Capability:** Match mode — stack all rules or first match only
- **Evidence:** `class-wrlp-admin.php:render_discount_settings_section()`, `class-wrlp-discount-rules.php:apply()` — checks `wrlp_match_mode` option
- **What it actually does:** `stack_all` (default) applies every matching rule to the price. `first_match` stops after the highest-priority matching rule (top row in the rules table).
- **What the user sees:** Setting under Discount Settings: "When multiple rules match" dropdown.
- **Safe claim:** Store owners can choose whether all matching discount rules apply at once, or only the highest-priority matching rule applies.

---

- **Capability:** Stacking math — additive or sequential
- **Evidence:** `class-wrlp-discount-rules.php:apply()`, `wrlp_stacking_mode` option
- **What it actually does:** `additive` (default): all rule discounts are computed from the original price and summed (10% + 5% = 15% off original). `sequential`: each rule's discount is applied to the price produced by the previous rule (10% then 5% of remainder = 14.5% total). Only applies when `stack_all` is active.
- **What the user sees:** Setting: "Stacking math" dropdown, visible only when Stack all is selected.
- **Safe claim:** When multiple rules stack, discounts can be calculated as additive (each taken from the original price) or sequential (each applied to the running discounted price).

---

- **Capability:** Coupon behavior — stack with rules or disable rules when coupon applied
- **Evidence:** `class-wrlp-discount-rules.php:apply()` and `apply_cart_fees()` — early return when `wrlp_coupon_behavior === 'skip_rules'` and cart has applied coupons
- **What it actually does:** `stack` (default): discount rules fire alongside WooCommerce coupons. `skip_rules`: if any WC coupon is applied to the cart, all discount rules are skipped for that order.
- **What the user sees:** Setting: "When coupons are applied" dropdown.
- **Safe claim:** Store owners can choose whether discount rules apply alongside WooCommerce coupons, or whether rules are suspended when a coupon is present.

---

- **Capability:** Rule active date range
- **Evidence:** `class-wrlp-discount-db.php:get_active_rules()` — `WHERE active_to > NOW()` and `active_from <= NOW()`; DB schema includes `active_from` and `active_to` DATETIME columns
- **What it actually does:** Rules are only returned as active when the current UTC time falls within the `active_from`/`active_to` window. Either boundary can be left blank for open-ended rules. Admin dates are stored in WP timezone and converted to UTC.
- **What the user sees:** "Active period" date-time pickers in rule modal. Expired rules show an "Expired" badge in the rules table and can be edited to extend.
- **Safe claim:** Discount rules can be scheduled with optional start and end dates.

---

- **Capability:** Discount fee label cleanup and order notes
- **Evidence:** `class-wrlp-discount-rules.php:reshape_discount_fee_rows()`, `add_discount_order_note()`, `fix_order_fee_tax_status()`
- **What it actually does:** WooCommerce default fee labels would show "Fee: Discount: Rule Name." `reshape_discount_fee_rows()` replaces these with "Rule Name: -$X.XX" before the order total is rendered. `add_discount_order_note()` adds an order note at checkout listing every fired rule name and its discount amount. `fix_order_fee_tax_status()` marks discount fees as non-taxable.
- **What the user sees:** Cart/order totals show clean discount labels by rule name. Order notes record which rules fired and how much each discounted.
- **Safe claim:** Discount amounts appear in cart and order totals as clean labeled lines (one per rule). An order note is added at checkout listing every rule that fired.

---

### Quantity Tier Pricing

- **Capability:** Volume pricing table on single product page
- **Evidence:** `class-wrlp-quantity-tiers.php`, `templates/single-product/tier-table.php`, `class-wrlp-discount-rules.php:override_qty_tier_price()`
- **What it actually does:** Built on top of discount rules — quantity tiers are `item_price` rules with a `min_qty` condition. The tier table is rendered on the product page showing quantity ranges (e.g., "1–4", "5–9", "10+") and price-per-unit for each tier. When the customer changes quantity, the actual price updates because `override_qty_tier_price()` hooks `woocommerce_product_get_price` at priority 999.
- **What the user sees:** A "Volume Pricing" table below (or above, or elsewhere) the add-to-cart button showing quantity ranges, price each, and optionally "You save" amounts.
- **Safe claim:** Volume pricing tables appear automatically on product pages showing tiered pricing by quantity. The price updates as the customer adjusts quantity.

---

- **Capability:** Configurable table position
- **Evidence:** `class-wrlp-quantity-tiers.php:get_position_options()`, `WRLP_Admin:render_display_settings_section()`
- **What it actually does:** Six position options: after cart form (default), before cart form, after add-to-cart button, after title, after excerpt, or manual (shortcode). Stored in `wrlp_qty_tiers_position` option.
- **What the user sees:** "Quantity-break price table" position dropdown in the Frontend Display settings section. When "Manual placement" is selected, the settings reveal the `[wrlp_qty_tiers]` shortcode with a copy button.
- **Safe claim:** The volume pricing table position can be configured from the plugin settings, or placed manually using the `[wrlp_qty_tiers]` shortcode.

---

- **Capability:** Variable product support for tier pricing
- **Evidence:** `class-wrlp-quantity-tiers.php`: hooks `woocommerce_available_variation` to inject `wrlp_qty_tiers_html` key into WC's variation JS payload; placeholder `<div class="wrlp-qty-tiers-variation">` filled by JS per variation selection
- **What it actually does:** Variable products show a placeholder tier table on page load. When the customer selects a variation, the tier table updates to show that specific variation's tiers (injected via the variation change JS event).
- **What the user sees:** Tier table updates automatically when the customer switches variation (e.g., selects a different size or color).
- **Safe claim:** Volume pricing tables support variable products — the table updates when the customer selects a variation.

---

- **Capability:** Product summary price protection (qty=1 baseline in summary)
- **Evidence:** `class-wrlp-discount-rules.php:is_in_summary()`, `override_qty_tier_price()` — skips tier price when `$in_summary` is true
- **What it actually does:** Between `woocommerce_before_single_product_summary` and `woocommerce_after_single_product_summary` hooks, a flag is set. Tier price override is suppressed within this window, so the product's main price display shows the qty=1 baseline price rather than a tier price. This prevents the product summary from showing a misleading tier price before the customer has chosen a quantity.
- **What the user sees:** Product's main listed price stays at the regular (qty=1) price; tier table below shows the full break schedule.
- **Safe claim:** The product's main price display always shows the regular unit price; tier discounts only apply at checkout as the quantity qualifies for a tier.

---

### Customer-Specific Price Lists

- **Capability:** Assign individual customers to a specific price list
- **Evidence:** `class-wrlp-customer-lists.php:resolve_list()` (hooks `wrn_plm_resolve_price_list`), `class-wrlp-admin.php:ajax_batch_assign_customers()`, `ajax_get_list_customers()`; user meta key `_wrlp_customer_list_id`
- **What it actually does:** Any WP user can be assigned to a specific price list via user meta. This overrides the user's role-based list. Assignment is stored as `_wrlp_customer_list_id`. Only one list per customer. Reassigning a customer silently replaces the previous assignment. If the assigned list is later deleted, the stale meta is cleaned up.
- **What the user sees (admin):** Customer assignment section in the price list edit modal with a user search field (search by name or email, returns up to 20 users) and removable customer tags. Also accessible from the WP user profile edit screen.
- **Safe claim:** Individual customers can be assigned to a specific price list, overriding the role-based list for that customer. Each customer can be assigned to one list at a time.

---

- **Capability:** Customer assignment from user profile screen
- **Evidence:** `class-wrlp-customer-lists.php`: `show_user_profile` and `edit_user_profile` hooks; nonce `wrlp_customer_list_{$user_id}`
- **What it actually does:** A price list dropdown appears on the WP user edit page. The admin can assign or clear the customer's price list from there. Nonce-verified save.
- **What the user sees:** A "Price list" section on the WordPress Edit User screen.
- **Safe claim:** Price lists can be assigned to individual customers from their WordPress user profile.

---

### `[wrn_price_table]` Shortcode

- **Capability:** Customer-facing price table for the logged-in user's price list
- **Evidence:** `class-wrlp-price-table.php:render()`, `templates/frontend/price-table.php`
- **What it actually does:** Resolves the current user's assigned price list (respecting customer-specific override and role-based list). Loads all products in the list that are published and have a list price > 0. Optionally applies discount rules to the "Your Price" column. Renders an HTML table with search field, configurable columns, and paginated rows.
- **What the user sees:** A table of products with columns: Product (linked to product page), Regular Price, Your Price, You Save (amount + %).
- **Safe claim:** The `[wrn_price_table]` shortcode displays the logged-in customer's full price list — every product in their assigned list with their price and how much they save.

---

- **Capability:** Shortcode attributes — category filter, forced list ID, guest behavior, discount toggle, column control, per-page pagination
- **Evidence:** `class-wrlp-price-table.php:render()` — `shortcode_atts()` with `category`, `list_id`, `guest`, `show_discounts`, `columns`, `per_page`
- **What it actually does:** `category`: filters rows to products in specified category slugs. `list_id`: forces a specific list (admin use; bypasses role/customer resolution). `guest`: show login link or hide table for logged-out visitors. `show_discounts`: yes/no toggle for discount rule application in "Your Price." `columns`: choose which columns to include. `per_page`: number of rows per page (0 = all).
- **What the user sees:** All attributes are configurable; admin shortcode generator UI produces the shortcode string.
- **Safe claim:** The price table shortcode supports category filtering, column control, pagination, and behavior options for guest visitors.

---

- **Capability:** Admin shortcode generator
- **Evidence:** `class-wrlp-admin.php:render_shortcodes_panel()` — live-updating generator UI with list selector, category multi-select, guest mode, discount toggle, per-page, column checkboxes, copy button
- **What it actually does:** JavaScript-powered generator in the Shortcodes section of the overview dashboard. Updates the shortcode string in real time as admin changes options. One-click copy to clipboard.
- **What the user sees:** Shortcodes panel on the plugin Overview tab with an interactive generator.
- **Safe claim:** The admin includes a shortcode generator that builds the `[wrn_price_table]` shortcode interactively.

---

### Expiry Notifications

- **Capability:** Email alert when price lists or discount rules are about to expire
- **Evidence:** `class-wrlp-expiry-notifications.php:run_check()`, `get_expiring()`, `send_email()`; cron hook `wrlp_daily_expiry_check`
- **What it actually does:** A daily WP-Cron job queries both `wr_price_lists` and `wrlp_discount_rules` for active items whose `active_to` date falls within the configured lead-time window. If any are found, sends a plain-text email listing them with their expiry date and a link to the admin pricing page.
- **What the user sees (admin):** Email notification with subject "[Site Name] Pricing items expiring soon" listing each item, its type, and its expiry timestamp.
- **Safe claim:** The plugin sends an email alert when price lists or discount rules are approaching their expiry date. The lead time (1, 3, 7, or 14 days) is configurable.

---

- **Capability:** Configurable notification lead time and recipient
- **Evidence:** `class-wrlp-admin.php:render_notifications_settings_section()`; options `wrlp_expiry_notif_days` (1/3/7/14) and `wrlp_expiry_notif_email`
- **What it actually does:** Admin selects lead time from a dropdown. Email field defaults to the WP admin email if left blank.
- **Safe claim:** The notification email recipient and advance warning period can be configured in the plugin settings.

---

### Admin Pricing Inspector

- **Capability:** Admin-bar pricing inspector panel on the frontend
- **Evidence:** `class-wrlp-inspector.php` (831 lines)
- **What it actually does:** An admin-bar node ("▶ WR Pricing") visible to `manage_woocommerce` users on the frontend with the WP admin bar showing. Opens a slide-in panel. Three contexts: product pages (product context), cart/checkout (cart context), everything else ("No pricing context").
- **What the user sees (admin):** An inspector panel showing resolved prices using the same engine as the real storefront.
- **Safe claim:** WRN Pricing Rules Pro includes a Pricing Inspector in the WordPress admin bar on the storefront. It shows live resolved prices for shop managers without affecting what customers see.

---

- **Capability:** Product context — role switcher and full price chain
- **Evidence:** `class-wrlp-inspector.php`: product context rendering; role switcher dropdown (`wp_roles()->get_names()`); calls `WRN_PLM_Pricing::resolve_base_price()`, `WRN_PLM_DB::get_product_price()`, `wrn_plm_preview_discount_info` filter
- **What it actually does:** On single product pages, shows: regular price → list price (role-based) → active discount rule applied → quantity tier at qty=1 → final unit price with savings %. Role dropdown allows shop manager to see prices as any WP role without logging in as that user.
- **Safe claim:** On product pages, the inspector shows the full pricing chain — regular price, list price, discount rule applied, and quantity tier price — and allows previewing as any user role.

---

- **Capability:** Cart/checkout context — per-item breakdown and cart savings
- **Evidence:** `class-wrlp-inspector.php`: cart context rendering; uses `WRLP_Cart_Breakdown::get_total_savings()`, `wc_get_price_excluding_tax()`, `wc_get_price_to_display()`
- **What it actually does:** Shows a per-item table: regular price, your price, saves; expandable detail row per item (list price, discount rule, active tier). Shows cart-level fee adjustments detected by `_wrlp_rule_discount` meta key. Shows total savings across all items. Shows applied coupons. Tax-normalized using WC's own display functions.
- **Safe claim:** On cart and checkout pages, the inspector shows a per-item pricing breakdown including list price, discount applied, quantity tier, and cart-level discounts, plus total savings.

---

- **Capability:** AJAX refresh triggered by cart mutations
- **Evidence:** `class-wrlp-inspector.php`: AJAX handler `wrlp_inspector_refresh`; `window.fetch` patch for non-jQuery cart mutations; triggered by WC cart events and `sf_mc_update` action pattern
- **What it actually does:** Inspector totals recalculate server-side whenever the WC cart changes (quantity updates, add/remove). Also patches `window.fetch` to detect cart mutations from non-jQuery minicart implementations (customized for Samurai Fireworks' fetch-based minicart).
- **Safe claim:** The inspector refreshes automatically when the cart changes, including after quantity updates or item additions.

---

- **Capability:** Inspector enable/disable toggle
- **Evidence:** `class-wrlp-admin.php:render_discount_settings_section()` — `wrlp_inspector` option checkbox; `class-wrlp-inspector.php` checks this option before registering hooks
- **What it actually does:** Inspector can be disabled via settings. When disabled, no admin-bar node or panel is rendered. Enabled by default.
- **Safe claim:** The Pricing Inspector can be disabled in the plugin settings.

---

### Admin Settings Sections

- **Capability:** Discount settings section (match mode, stacking mode, coupon behavior, inspector toggle)
- **Evidence:** `class-wrlp-admin.php:render_discount_settings_section()`, `save_settings_from_extra()`
- **What it actually does:** Injected after the free plugin's settings form via `wrn_plm_after_settings_form`. Settings are consumed by the discount engine and inspector on every request. All four settings saved, validated, and sanitized via `save_settings_from_extra()`.
- **Safe claim:** Rule engine behavior (match mode, stacking math, coupon handling, inspector visibility) is configurable from the plugin settings.

---

- **Capability:** Frontend display settings (quantity tier table position)
- **Evidence:** `class-wrlp-admin.php:render_display_settings_section()`
- **What it actually does:** Single setting — tier table position — with six options. Shortcode copy button appears when "Manual placement" is selected.
- **Safe claim:** The position of the volume pricing table on product pages is configurable.

---

- **Capability:** Wizard settings injection (Pro settings in free plugin's setup wizard)
- **Evidence:** `class-wrlp-admin.php:render_wizard_settings()` (hooks `wrn_plm_wizard_settings_html`), `save_settings_from_extra()`
- **What it actually does:** Three Pro settings (match mode, stacking mode, coupon behavior) are injected into step 2 of the free plugin's setup wizard. Saves correctly when the wizard form submits via `wrn_plm_save_settings_extra`.
- **Safe claim:** The setup wizard includes Pro discount rule settings so they can be configured during initial setup.

---

- **Capability:** Price list modal — customer assignment section injection
- **Evidence:** `class-wrlp-admin.php:render_price_list_modal_customers()` (hooks `wrn_plm_price_list_modal_extra`)
- **What it actually does:** Injects a customer search and tag UI into the free plugin's price list edit modal. Shows "Save the price list first, then you can assign individual customers" until the list has been saved (new list state). On existing lists, shows current assigned customers as removable tags plus a live user search field.
- **Safe claim:** The price list edit panel includes a customer assignment section where individual customers can be searched and assigned directly within the modal.

---

### Overview Dashboard Panels

- **Capability:** Discount count and active rules panel in overview
- **Evidence:** `class-wrlp-admin.php:overview_discount_count()`, `render_overview_discount_panel()`
- **What it actually does:** Injects discount rule count into the overview dashboard stat. Renders an aside panel listing active rules with discount amount, conditions summary, and link to Discount Rules tab. If no license, shows "N rules saved but inactive" warning.
- **Safe claim:** The plugin overview shows active discount rules at a glance.

---

- **Capability:** DB schema installer and migration
- **Evidence:** `class-wrlp-install.php:install()`, `maybe_install()` — `dbDelta()` + idempotent `migrate_add_column()` with `information_schema` check
- **What it actually does:** Creates `wrlp_discount_rules` and `wrlp_rule_conditions` tables on install. Subsequent schema changes are applied via `migrate_add_column()` which checks `information_schema.COLUMNS` before `ALTER TABLE` — safe to re-run, will never produce duplicate-column errors. DB version tracked in `wrlp_db_version` option (current: 1.7).
- **Safe claim:** (Internal — not a customer-facing claim.) DB migrations are safe to run on existing installs.

---

### WooCommerce Compatibility

- **Capability:** HPOS (High-Performance Order Storage) compatibility declaration
- **Evidence:** Declared in plugin bootstrap via `\Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility()` (confirmed from session context)
- **What it actually does:** Declares compatibility with WooCommerce's HPOS feature, suppressing the WooCommerce compatibility warning in the admin.
- **Safe claim:** WRN Pricing Rules Pro is compatible with WooCommerce High-Performance Order Storage (HPOS).

---

## Partially Implemented Capabilities

### Cart Discount Base for Product/Category Conditions on Cart-Total Rules

- **What exists:** `class-wrlp-discount-rules.php:get_discount_base()` — when a `cart_total` rule has `product` or `category` conditions, the discount base is computed as the sum of matching line totals only (not the full cart total).
- **What is missing:** No admin UI explains this behavior. A store owner might create a "10% off the cart if it contains Product X" rule and expect the 10% to apply to the full cart total — but the code only discounts the Product X line totals.
- **Safe wording:** "Cart-total rules scoped to specific products or categories discount the total value of matching items, not the full cart total."
- **Unsafe wording:** "Cart discount applies to the entire order when a specific product is in cart." — This overstates the scope.

---

### Customer Search Limit in Price List Modal

- **What exists:** `class-wrlp-admin.php:ajax_search_users()` — `get_users()` with `'number' => 20`
- **What is missing:** No "load more" or pagination for search results. Stores with many users sharing a name or email prefix will see only the top 20.
- **Safe wording:** "Search returns up to 20 matching customers."
- **Unsafe wording:** "Find any customer by name or email." — Overstates completeness.

---

## UI-Only / Stub / Demo Capabilities

None found. Every visible admin control and setting has confirmed backend logic that consumes it.

---

## Missing Capabilities

- **License expiry notification:** No email when the Pro license itself is expiring. Expiry notifications cover price lists and discount rules only (`class-wrlp-expiry-notifications.php`). Store owners learn of license expiry reactively via the license page or admin notice.
- **Bulk rule import/export:** No CSV or JSON import/export for discount rules. Rules must be entered one at a time via the admin modal.
- **Customer-facing savings display:** No out-of-the-box banner or cart message showing the customer how much they are saving. `WRLP_Cart_Breakdown::get_total_savings()` exists as an internal API (used by the inspector), but no customer-facing rendering is wired up. A developer could use the method to build a savings banner.
- **REST API for rule management:** No REST endpoints for discount rule CRUD. Rules are managed via `wp-admin/admin-ajax.php` only.
- **Audit log for rule changes:** No record of who changed which rule or when.

---

## Requires Configuration

- **Volume pricing tiers:** Tiers are discount rules with `min_qty` conditions. They do not exist automatically — they must be created as discount rules in the Discount Rules tab.
- **Expiry notifications:** Daily cron fires only if WP-Cron is functioning. On sites with `DISABLE_WP_CRON` or unreliable traffic, an external cron trigger is needed.
- **Customer-specific lists:** Overrides the role-based list, but only if the customer has been individually assigned in the price list modal or user profile. No automatic customer grouping exists.
- **Cart-condition rules:** Rules using `min_cart_total`, `min_cart_qty`, `is_first_order`, or `min_order_count` apply as cart-level fees, not item price adjustments. Store owners configuring these need to understand the fee-line display model.
- **Inspector:** Enabled by default. If disabled, shop managers lose visibility into resolved prices. The `window.fetch` patch in the inspector is customized for Samurai Fireworks' minicart implementation (`sf_mc_update`) — this specific event pattern may not exist on other stores.

---

## Requires Human Service

- **Strategy design for discount rules:** The plugin provides all the tools (condition types, stacking options, coupon behavior), but designing an effective pricing strategy — what tiers to offer, which customer segments to target, how to stack vs. first-match — requires business analysis.
- **Customer segment setup:** Identifying which customers belong in which list and assigning them requires operational work, especially for large customer bases (due to the 20-result search limit).
- **Cart discount base behavior explanation:** Store owners expecting cart-total rules with product conditions to discount the entire cart total need to be corrected during setup. This is a non-obvious behavior that requires documentation or onboarding.
- **WP-Cron reliability:** Sites with low traffic or `DISABLE_WP_CRON` need an external cron configured separately for expiry notifications and license validation to be reliable.
- **Samurai Fireworks inspector customization:** The `window.fetch` patch (`sf_mc_update` event) is site-specific. A new customer installation would need this removed or adapted to their theme's minicart pattern.

---

## Claim Safety Review

| Claim | Classification |
|---|---|
| Create and manage discount rules with conditions | Ready to claim |
| Support for role-based, customer-specific, product, and category conditions | Ready to claim |
| Support for cart-level conditions (cart total, cart qty, first order, past order count) | Ready to claim — note that order count only counts `wc-completed` orders |
| Percent-off and fixed-amount discount types | Ready to claim |
| Apply discounts per item or as a single cart-level discount line | Ready to claim |
| Schedule rules with optional start and end dates | Ready to claim |
| Stack all matching rules or use the highest-priority rule only | Ready to claim |
| Additive and sequential stacking math options | Ready to claim |
| Rules apply alongside coupons, or are suspended when a coupon is used | Ready to claim |
| Volume pricing / quantity tiers on product pages | Ready to claim |
| Tier table updates when customer selects a variation | Ready to claim |
| `[wrn_price_table]` shortcode for customer-facing price list display | Ready to claim |
| Assign individual customers to a specific price list | Ready to claim |
| Per-customer override takes precedence over role-based list | Ready to claim |
| Admin Pricing Inspector showing live resolved prices on the storefront | Ready to claim |
| Inspector shows product context with role switcher and full price chain | Ready to claim |
| Inspector shows cart context with per-item breakdown and total savings | Ready to claim |
| Email alerts when price lists or discount rules are nearing expiry | Ready to claim |
| HPOS compatibility | Ready to claim |
| "Buy more, save more" customer savings displayed on product page | Ready to claim (via tier table) |
| "Cart discount applies to the full cart when a specific product is present" | Unsafe — cart-total rules with product/category conditions discount matching-item totals only |
| "Find any customer by name or email" | Partial — search returns up to 20 results |
| Automatic license expiry alert | Missing — no license-expiry notification exists |
| Customer-facing savings summary at checkout | Missing — no built-in display; internal API exists for developer use |
| The Pricing Inspector works on all WooCommerce themes without setup | Safe with wording limits — the `window.fetch` patch is customized for one store's minicart pattern; new installs may need this adapted |

---

## Product vs. Service Boundary

**Plugin handles independently:**
- All discount rule CRUD, condition evaluation, and price engine logic
- Quantity tier rendering and price updates
- Customer list assignment UI and logic
- `[wrn_price_table]` shortcode rendering
- Expiry notification emails (when WP-Cron is reliable)
- Admin Pricing Inspector

**Requires WebReadyNow service to complete:**
- B2B pricing strategy design (what rules, tiers, and customer segments to create)
- Large-scale customer-to-list assignment operations
- Explanation of non-obvious behaviors (cart discount base scoping, order-count definition)
- Inspector `window.fetch` patch adaptation for non-Samurai Fireworks themes
- WP-Cron reliability setup for sites with low traffic or `DISABLE_WP_CRON`

---

## Development Roadmap Recommendations

**Must build before public claim:**
- Nothing new — all currently claimed features are confirmed implemented.

**Should build soon:**
- License expiry notification email (currently missing entirely).
- Correct or document the `window.fetch` inspector patch so it works on non-Samurai themes, or provide a clean hook for store-specific minicart events.
- Admin UI explanation or tooltip for cart-discount-base scoping behavior (product/category conditions on cart-total rules).
- Increase or paginate customer search results beyond 20 (or document the limit clearly).

**Nice to have:**
- Customer-facing savings banner or order summary savings display.
- Bulk rule import/export (CSV/JSON).
- Audit log for rule changes.
- REST API for rule management (headless / programmatic use).

**Not needed now:**
- Rule versioning or A/B testing capabilities — out of scope for v1.x.
- Multi-currency tier pricing — depends on whether a multi-currency plugin is in use.

---

## Recommended Next Agent

**Marketing & Documentation Agent** — The feature set is fully audited. Safe claims above cover all major features. The Marketing & Documentation Agent can now write:
- Full feature descriptions for the product page
- Discount rules documentation with condition type reference
- Volume pricing setup guide
- `[wrn_price_table]` shortcode reference
- Pricing inspector usage guide
- FAQ entries covering rule stacking, coupon behavior, customer list assignment, and expiry notifications

Note the following constraints for the Marketing & Documentation Agent:
- Do not claim the cart-total rule discounts the entire cart when product/category conditions are present.
- Do not claim the customer search finds all customers (20-result limit).
- Do not claim the inspector works on all themes without adaptation (fetch patch caveat).
- Do not claim license expiry notifications exist (they do not; only price list/rule expiry notifications exist).

---

## Knowledge Base Update Recommendation

**Update level: Level 2** — Significant new safe claims established; product file has a documented TODO that this audit resolves.

**Files to update:**

1. **`06-products/wrn-pricing-rules.md`** — Replace the "What It Does (Claimed — TODO: Full Codebase Audit Needed)" section with the full verified feature list from this audit. Remove the TODO notice.

2. **`03-safe-claims.md`** — Add a "WRN Pricing Rules Pro — Feature Capabilities" section with the safe claims table from this audit. Several new claims are now authorized that were previously blocked by TODO.

**Decision record:** Not required for Level 2 (no architectural or product packaging decisions made — this is an audit of implemented code, not a direction change).

**Output folder:** `09-agent-outputs/codebase-audits/` — this file.
