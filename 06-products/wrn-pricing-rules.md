# WRN Pricing Rules / Price List Manager — Product File

**Purpose:** Load this file when working on WRN Pricing Rules code, marketing, or B2B pricing services.

---

## Product Identity

**Names:** WRN Pricing Rules Pro / WRN Pricing Rules Free / WR Price List Manager
**Type:** WooCommerce B2B and wholesale pricing plugin
**Versions:** Pro (paid) + Free (lite, WP.org or direct)
**Status:** Active

---

## What It Does — Verified Feature Set

**Status:** Full codebase audit completed 2026-06-29. See `09-agent-outputs/codebase-audits/wr-price-list-pro-feature-capability-audit-v1.2.1.md`.

### Free Plugin (wr-price-list-for-woocommerce)
- Wholesale and role-based pricing in WooCommerce
- B2B / B2C pricing separation
- Price lists assigned by user role
- VIP and distributor pricing workflows

### Pro Plugin (wr-price-list-pro) — Implemented and Audited

**Discount Rules Engine**
- Create, edit, delete, reorder, and enable/disable individual rules
- Discount types: percent off, fixed amount off
- Apply to: per-item (adjusts product price) or cart total (single fee line in order totals)
- Active date range: optional start and end dates per rule
- All conditions on a rule use AND logic; no conditions = applies to all users and products

**Condition Types (9 types):**
- `role` — user's WP role
- `customer` — specific customer (user ID)
- `product` — one or more specific products or variations
- `category` — product category
- `min_qty` — minimum item quantity in cart
- `min_cart_total` — cart subtotal meets threshold
- `min_cart_qty` — total cart item count meets threshold
- `is_first_order` — customer has zero completed orders
- `min_order_count` — customer has at least N completed orders

**Wording constraint — order count:** `is_first_order` and `min_order_count` count only `wc-completed` orders. Pending and processing orders do not count.

**Rule Stacking and Behavior**
- Match mode: stack all matching rules, or first match only (top-priority rule wins)
- Stacking math (when stacking): additive (10% + 5% = 15% off original) or sequential (10% then 5% of remainder = 14.5% total)
- Coupon behavior: stack with rules, or disable rules when a WooCommerce coupon is applied

**Wording constraint — cart discount base:** Cart-total rules with product or category conditions apply the discount to matching-item totals only, not the full cart total.

**Volume Pricing (Quantity Tiers)**
- Volume pricing tables built from discount rules using `min_qty` conditions
- Table displays on single product pages with quantity ranges and price-per-unit
- Configurable position: after cart form (default), before cart form, after add-to-cart button, after title, after excerpt, or manual placement via shortcode
- Variable product support: table updates per-variation when customer selects variation
- `[wrlp_qty_tiers]` shortcode for manual placement
- Product summary shows qty=1 baseline price; tier prices apply at cart level by quantity

**`[wrn_price_table]` Shortcode**
- Displays logged-in customer's full price list as a searchable, paginated table
- Columns: Product, Regular Price, Your Price, You Save (each individually toggleable)
- `show_discounts` attribute: apply or exclude discount rules from the "Your Price" column
- `category` attribute: filter rows by product category slug
- `list_id` attribute: force a specific list (admin/developer use)
- `guest` attribute: show login link or hide table for logged-out visitors
- `per_page` attribute: rows per page (0 = all)
- Admin shortcode generator: interactive builder in the plugin overview that outputs the ready-to-paste shortcode

**Customer-Specific Price Lists**
- Assign individual customers to a specific price list (overrides role-based list)
- Assignment stored as `_wrlp_customer_list_id` user meta
- Each customer is assigned to one list at a time (reassigning replaces previous)
- Assignment UI: price list edit modal (with customer search and removable tags) and WP user profile screen
- Stale meta auto-cleaned when a list is deleted

**Wording constraint — customer search:** Customer search in the modal returns up to 20 results. Do not claim unlimited customer search.

**Admin Pricing Inspector**
- Admin-bar panel ("▶ WR Pricing") visible to `manage_woocommerce` users on the storefront
- Product context: shows regular price → list price → discount rule applied → quantity tier → final unit price with savings %; role switcher to preview as any WP role
- Cart/checkout context: per-item pricing breakdown (list price, discount rule, tier), cart-level fee adjustments, total savings, applied coupons
- AJAX refresh when cart changes (quantity updates, add/remove)
- Enable/disable via settings toggle

**Wording constraint — inspector:** The inspector's `window.fetch` patch is customized for Samurai Fireworks' minicart implementation. On other stores, this event listener may need theme-specific adaptation.

**Expiry Notifications**
- Daily cron sends email when price lists or discount rules are approaching their expiry date
- Configurable lead time: 1, 3, 7, or 14 days
- Configurable notification email (defaults to WP admin email)

**Wording constraint — expiry notifications:** These notifications cover price list and discount rule expiry only. There are no license expiry notifications.

**Discount Display and Order Records**
- Cart/order totals show discount as clean labeled lines (one per rule, named after the rule)
- Order note added at checkout listing every rule that fired and how much it discounted
- Discount fees are non-taxable

**Admin Settings Sections**
- Discount settings: match mode, stacking math, coupon behavior, inspector toggle
- Frontend display settings: quantity tier table position
- Notifications settings: expiry lead time, notification email
- Setup wizard: Pro discount settings injected into free plugin's wizard step 2

**WooCommerce Compatibility**
- HPOS (High-Performance Order Storage) compatibility declared

---

## Offer Models

| Model | Description |
|---|---|
| Plugin-only (Free) | WP.org or direct download — limited feature set |
| Plugin-only (Pro) | Paid license — full feature set |
| Plugin + Setup | Pro license + WebReadyNow configuration session |
| B2B Pricing Implementation | Full service — design and implement B2B pricing strategy in WooCommerce |
| Custom pricing workflow | Custom development for complex pricing requirements |

---

## Who Buys It

- B2B WooCommerce stores
- Wholesalers and distributors
- Manufacturers selling direct to multiple buyer types
- Stores needing VIP or customer-specific pricing
- Stores that need to show different prices to logged-in vs guest users

---

## Proof

- WR Price List Manager case study — published at webreadynow.com
- This product was built for a real client and later productized
- Supports the Custom WooCommerce Plugin Development service narrative

See `08-proof/case-study-index.md` for details.

---

## Published Content Supporting This Product

- How to Offer Wholesale Prices in WooCommerce Without a Separate Store
- Price Lists vs Discounts in WooCommerce: Which One Does Your Store Actually Need?
- WooCommerce Role-Based Pricing: The Complete Guide for B2B Stores
- VIP Member Pricing article (title TODO — confirm exact title)

---

## Safe Claims

**Free plugin:**
- Supports wholesale and role-based pricing in WooCommerce
- Supports B2B/B2C pricing separation
- Supports customer-specific price lists
- Supports VIP and distributor pricing

**Pro plugin (audited 2026-06-29 — see "What It Does" section above for full detail):**
- Advanced discount rules based on role, customer, product, category, quantity, cart totals, cart quantity, first order status, and past order count
- Percent-off and fixed-amount discounts, applied per item or as a single cart-line fee
- Configurable rule stacking and coupon behavior
- Volume pricing tables with configurable placement and variable product support
- `[wrn_price_table]` shortcode with filtering, column control, pagination, and shortcode generator
- Individual customer-to-price-list assignment overriding role-based pricing
- Pricing Inspector for reviewing product and cart pricing as any user role
- Expiry notifications for price lists and discount rules
- HPOS compatibility

See `03-safe-claims.md` → "WRN Pricing Rules Pro — Feature Claims" for approved wording, wording limits, and unsafe claims.

---

## Pro License and Update Integration (WRN Hub)

**Status:** Audited and launch-ready as of 2026-06-29. See `09-agent-outputs/codebase-audits/wr-price-list-pro-wrn-hub-integration-v1.2.1.md`.

**Infrastructure:** WRN Hub (on webreadynow.com) is the license server, update server, and download server for WRN Pricing Rules Pro. It is not installed on customer sites.

**How the integration works:**

- The Pro plugin validates the license key against the WRN Hub REST API (`wrn-hub/v1/license/validate`) daily via WP-Cron.
- Pro features gate behind `WRLP_License::is_valid()`. They activate when the license is valid and deactivate when confirmed invalid.
- Plugin updates appear in the WordPress Plugins screen via WP's native update transient injection. Update checks hit `wrn-hub/v1/update/wr-price-list-pro` (public endpoint, no auth required).
- Plugin downloads during updates are authenticated: `wrn-hub/v1/download/wr-price-list-pro?key={license}`.

**Customer-facing behavior:**

- Customers retrieve license keys from webreadynow.com/my-account — not from email.
- The Pro License admin page shows status, plan, renewal date, and activated site count.
- Expired, suspended, revoked, and invalid states each show a specific customer-facing message with a link to My Account.
- An admin notice fires on WRN Pricing admin pages when the license is confirmed invalid so the store owner is not left in the dark.
- A 7-day grace period keeps Pro features active if the license server is temporarily unreachable.

**Known boundaries and requirements:**

- The free `wr-price-list-for-woocommerce` plugin must be installed and active — the Pro plugin will not load without it.
- WP-Cron must be functioning for daily license re-validation and expiry notifications to fire.
- Outbound HTTPS requests to webreadynow.com must be reachable from the customer site.
- Deactivation slot cleanup depends on the license server being reachable at the moment of deactivation. Ghost activations can accumulate if the API call fails silently.
- License expiry notifications are not yet implemented at the license level (only at the price list and discount rule expiry level).
