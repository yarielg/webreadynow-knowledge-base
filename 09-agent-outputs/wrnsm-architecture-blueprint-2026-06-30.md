# WRN Store Monitor — Evidence Intelligence Architecture Blueprint
*Produced: 2026-06-30 | Plugin version at research time: v1.16.3*

---

## Preface

This document guides the next major development phase of WRN Store Monitor. It was produced after a live incident at Samurai Fireworks where:
- 80+ false-positive checkout errors accumulated (block checkout React hydration state)
- Free-form AI SQL queries caused unsafe DB access on a production store
- The AI correctly identified `pickup_location0` appearing in payment context but lacked structured evidence to confirm it

The core architectural rule going forward: **the AI never accesses the database directly.** All evidence is assembled by controlled, named, read-only PHP tools that return sanitized, structured arrays.

---

## 1. Evidence Intelligence Blueprint

### 1.1 Checkout Errors (Block and Classic)

| Field | Detail |
|---|---|
| **Evidence needed** | Error message text (normalized), checkout type (block/shortcode), payment method slug at time of error, time on page, order total range, error frequency over time, whether error is user-visible vs transient render state |
| **Safely collectable by plugin** | `wrnsm_events` table rows where `event_type = 'checkout_error'`; checkout_type from `WRNSM_Checkout_Tracker::detect_checkout_type()`; error message grouped by normalized text; gateway slug from event meta; time-on-page (seconds, no IP); frequency patterns (first/last occurrence, count in 24h/7d) |
| **Requires server/gateway access** | Whether a payment actually processed server-side for the same cart; gateway dashboard error codes; server-side 4xx/5xx logs from the checkout POST endpoint |
| **Never collect** | IP addresses; session tokens; customer name/email/phone; raw cart contents with pricing details; credit card last-4; CSRF nonces; raw form POST body |
| **How AI should use it** | Cluster errors by normalized message. If same message > 3x in 24h = pattern. If block checkout + "no payment methods" and count spikes at page-load time = transient hydration noise. If "payment declined" + specific gateway = gateway signal. If "shipping method not available" or "invalid payment method" = likely config issue. AI must not conclude "checkout is broken" from a single error; it needs a pattern. |

### 1.2 Payment Failures and Declines

| Field | Detail |
|---|---|
| **Evidence needed** | Per-gateway success rate over 7 and 30 days; failure count; failure velocity (is it accelerating?); whether failures started on a specific date; whether a gateway setting changed near failure start; order statuses for failed payments; error strings from order notes (pattern-matched, not raw) |
| **Safely collectable by plugin** | `WRNSM_Payment_Health::get_gateway_stats(7/30)` — aggregate counts, success rate, last failure timestamp; `wrnsm_events` payment_failed rows with gateway slug and anonymized amount range; order status distribution from `wc_get_orders()`; gateway change events from event log (enabled/disabled timestamps); checkout error event clusters correlated with gateway |
| **Requires server/gateway access** | Stripe/PayPal/gateway dashboard decline codes (e.g., `insufficient_funds`, `do_not_honor`, `card_velocity_exceeded`); webhook delivery logs; gateway account status (live vs test, restricted); 3DS authentication results |
| **Never collect** | Card numbers, last-4, expiry; customer billing address; gateway secret/publishable keys; raw Stripe/PayPal API responses |
| **How AI should use it** | Success rate < 90% = signal. Sudden drop on specific date + gateway change event nearby = strong correlation. All failures on one gateway + other gateways OK = gateway-specific issue. Across all gateways = store-wide issue (checkout page, SSL, caching). AI should rank gateway rate data highest; order note patterns as supporting. Never conclude card fraud or bank-side issue without decline codes from gateway dashboard (human layer). |

### 1.3 Stuck Orders (Pending Payment, Processing, On-Hold)

| Field | Detail |
|---|---|
| **Evidence needed** | Count of orders per problematic status; how long they have been stuck (age distribution); which gateway they were placed through; whether WP-Cron/Action Scheduler is running; whether payment capture hooks are registered; whether inventory was held |
| **Safely collectable by plugin** | `wc_get_orders()` aggregate by status (HPOS-compatible); order age buckets (< 1h, 1–24h, 1–7d, > 7d) — no PII; which gateway IDs appear in stuck orders (meta key `_payment_method` or HPOS equivalent); WP-Cron and Action Scheduler status from existing monitors; `wrnsm_events` for gateway_change events near stuck order start date |
| **Requires server/gateway access** | Whether payment actually captured on gateway side; webhook delivery status; manual order verification in gateway dashboard |
| **Never collect** | Customer name/email; order items with pricing; shipping address; order notes raw text |
| **How AI should use it** | All stuck in `pending` + cron failing = likely cron/webhook delivery issue. All stuck in `processing` + specific gateway = likely capture issue or async payment flow. Mix of statuses = broader problem. AI should flag stuck age > 24h as high-signal. Never conclude "orders are lost revenue" without human gateway verification. |

### 1.4 Invalid Payment Methods / No Payment Methods Available

| Field | Detail |
|---|---|
| **Evidence needed** | Which gateways are registered; which are marked enabled in WC settings; which are returned by `WC()->payment_gateways()->get_available_payment_gateways()` in admin context; whether any return empty or non-string gateway IDs; whether shipping/pickup methods are leaking into payment method list; checkout type; `woocommerce_available_payment_gateways` filter hooks registered by active plugins |
| **Safely collectable by plugin** | Registered vs available gateway count and ID list; any gateway returning empty ID; plugins hooking `woocommerce_available_payment_gateways`; checkout error events with "no payment methods" text; checkout_type from detector; Store API `/wc/store/v1/checkout` availability probe |
| **Requires server/gateway access** | Whether gateway API credentials are valid (test/live key mismatch — only discoverable via gateway API call); whether gateway account is restricted on provider side |
| **Never collect** | API keys, secret keys, webhook secrets; publishable keys |
| **How AI should use it** | Zero available gateways in WC admin but gateways enabled = filter removing them. Specific plugin hooking `woocommerce_available_payment_gateways` = prime suspect. Non-string or empty gateway ID = registration bug in gateway plugin. Pickup methods in payment list = shipping plugin not respecting payment registration boundaries (Samurai pattern). AI can name the suspect plugin/filter but cannot confirm without human inspection. |

### 1.5 Webhook Failures

| Field | Detail |
|---|---|
| **Evidence needed** | Whether any Action Scheduler jobs related to gateway webhooks are failing; order meta indicating webhook was received vs not; whether orders are stuck in states that should transition on webhook receipt; whether the WooCommerce REST API and site URL are publicly reachable |
| **Safely collectable by plugin** | Action Scheduler job counts by hook name pattern and status (pending/complete/failed/in-progress) for gateway-related hooks; checkout probe HTTP status of `/?wc-api=` endpoints [verify reachability]; orders stuck in gateway-specific pending states; `wrnsm_events` for `gateway_change` events that might indicate a key rotation triggering webhook secret mismatch |
| **Requires server/gateway access** | Gateway dashboard webhook delivery logs (Stripe: Events → Webhooks; PayPal: Activity → Webhooks); server firewall rules blocking inbound POST to `/?wc-api=`; SSL certificate validity on webhook endpoint |
| **Never collect** | Webhook signing secrets; raw webhook payload body; IP address of webhook sender |
| **How AI should use it** | AS jobs failing with gateway webhook hook patterns = webhook processing issue. Orders stuck in gateway-specific status that should auto-update = webhook not received or not processed. Recent gateway_change event = possible key rotation = webhook secret mismatch. AI should recommend human gateway dashboard check; cannot confirm without external access. |

### 1.6 WP-Cron / Action Scheduler Failures

| Field | Detail |
|---|---|
| **Evidence needed** | Whether WP-Cron is enabled and overdue; whether Action Scheduler is installed and processing; failed/pending job counts by group and hook; job age (oldest pending job); whether `DISABLE_WP_CRON` is set; whether a real cron job is configured as alternative |
| **Safely collectable by plugin** | Existing `WRNSM_Monitor_Cron` and `WRNSM_Monitor_Action_Scheduler` results; `DISABLE_WP_CRON` constant check; last cron run estimate via `wp_next_scheduled()`; AS job counts by status and hook name; last AS run timestamp |
| **Requires server/gateway access** | Actual server cron tab entries; server load causing cron timeouts; memory limits hitting mid-cron |
| **Never collect** | Job arguments containing user data or tokens; raw job output |
| **How AI should use it** | DISABLE_WP_CRON = true + no real cron evidence = cron dead. AS jobs piling up in `pending` = cron not running. AS jobs in `failed` on gateway hooks = combined cron + gateway issue. Never claim cron is "fixed" — only note what the evidence suggests. |

### 1.7 Performance Degradation

| Field | Detail |
|---|---|
| **Evidence needed** | Checkout page response time trend (multiple probes); cart page response time; WC-AJAX probe response time; PHP version; object cache presence; debug mode status; active plugin count; theme; whether slow requests correlate with a specific time of day |
| **Safely collectable by plugin** | `WRNSM_Checkout_Prober` last result (status, http_status, response_ms, checked_at); cart probe response time; AJAX probe response time; environment scan (PHP version, object cache, WP_DEBUG status); active plugin count from diagnostic payload; checkout probe trend from `wrnsm_scan_history` snapshots (last 30 scans) |
| **Requires server/gateway access** | Server-side slow query logs; PHP-FPM worker pool status; CDN or hosting panel metrics; APM tool (New Relic, Datadog) data |
| **Never collect** | Server credentials; hosting panel login details; raw database slow query log |
| **How AI should use it** | Response > 3000ms = degraded. Trend worsening = progressive issue. Correlates with high plugin count = possible conflict or resource drain. Object cache absent = PHP memory pressure risk. No caching → high TTFB is expected pattern. AI can suggest investigation areas; cannot confirm root cause without server access. |

### 1.8 Plugin / Theme Conflicts Affecting Checkout or Orders

| Field | Detail |
|---|---|
| **Evidence needed** | List of active plugins; which plugins hook into `woocommerce_checkout_*`, `woocommerce_payment_*`, `woocommerce_available_payment_gateways`; recently activated/deactivated plugins near incident start; theme name and version; whether any plugin modifies `$_POST` at checkout |
| **Safely collectable by plugin** | Active plugin slugs and versions from diagnostic payload; plugin lifecycle events from `wrnsm_events` (activated/deactivated with timestamps); `$wp_filter` inspection for specific hooks (return hook count per plugin file, not raw code); checkout error correlation with plugin activation dates |
| **Requires server/gateway access** | Code-level filter inspection (which exact function is registered); deactivation testing (requires staging); PHP error log entries from plugins |
| **Never collect** | Plugin license keys or activation tokens stored in plugin options; raw PHP error log lines containing customer data |
| **How AI should use it** | Plugin activated N days ago + checkout errors started N days ago = strong correlation. Plugin hooking `woocommerce_available_payment_gateways` + "no payment methods" error = prime candidate. AI should name the correlation clearly but recommend deactivation testing on staging — never recommend deactivating on production without backup. |

### 1.9 Email Delivery Failures

| Field | Detail |
|---|---|
| **Evidence needed** | Whether an SMTP plugin is active; WooCommerce email settings (from address, from name); whether WC critical emails are enabled; whether any transactional email has recently failed (not directly observable without SMTP plugin log access); order status events where email should have fired |
| **Safely collectable by plugin** | Existing `WRNSM_Monitor_Email` results (SMTP plugin detected, from address configured, WC email status); whether `wp_mail()` has been overridden by SMTP plugin; order count in statuses that trigger confirmation emails (aggregate, no PII) |
| **Requires server/gateway access** | SMTP plugin delivery logs (FluentSMTP, WP Mail SMTP, Postmark, SendGrid); MX record configuration; SPF/DKIM/DMARC validation; server mail queue status |
| **Never collect** | Customer email addresses; email subjects (may contain order numbers); raw email body; SMTP password |
| **How AI should use it** | No SMTP plugin = using PHP mail() = high deliverability risk. From address not matching domain = SPF mismatch likely. AI can recommend SMTP plugin or check SPF records; cannot confirm delivery without SMTP log access (WebReadyNow service layer). |

### 1.10 Inventory and Stock Issues

| Field | Detail |
|---|---|
| **Evidence needed** | Whether stock management is enabled; count of out-of-stock products; whether backorders are allowed; whether any product went out of stock recently near an order spike; whether WC stock reduction logic ran on affected orders |
| **Safely collectable by plugin** | Existing `WRNSM_Monitor_Stock` results (low stock count, out-of-stock count, stock management enabled); order status distribution; `wrnsm_events` if stock-related events are logged [currently not logged — gap]; whether `woocommerce_reduce_order_stock` hook is being called for stuck orders |
| **Requires server/gateway access** | Inventory sync system (ERP, ShipStation, custom integration) logs; warehouse stock counts; whether stock discrepancy is caused by integration lag |
| **Never collect** | Product prices or margin data; supplier information; raw inventory import logs |
| **How AI should use it** | Out-of-stock count spike = possible inventory issue or high demand event. Low stock threshold alerts = informational. AI cannot determine cause of stock discrepancy without integration access; recommend manual inventory audit if counts look wrong. |

---

## 2. Gateway Intelligence Matrix

> Fields marked `[verify]` should be confirmed against the plugin source before implementing specific detectors. The information below is based on known plugin behavior patterns as of mid-2025.

---

### 2.1 WooPayments (by Automattic)

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-payments` |
| **Gateway IDs** | `woocommerce_payments` (primary); `woocommerce_payments_giropay`, `woocommerce_payments_sepa`, `woocommerce_payments_sofort`, `woocommerce_payments_p24`, `woocommerce_payments_ideal`, `woocommerce_payments_becs`, `woocommerce_payments_eps`, `woocommerce_payments_bancontact` (APM sub-gateways) |
| **Log sources** | WC logger channel: `woocommerce-payments`; custom DB tables: `wcpay_payment_intents`, `wcpay_charges`, `wcpay_refunds`, `wcpay_customers` |
| **Webhook indicators** | `_wcpay_payment_intent_id` in order meta; `_wcpay_payment_intent_status` (succeeded/requires_capture/cancelled); webhook events processed via custom DB tables |
| **Action Scheduler hooks** | `wcpay_track_new_invoice` [verify]; subscription renewal hooks [verify]; `wcpay_process_payment_with_checkout_rate_limit` [verify] |
| **Safe order meta fields** | `_wcpay_payment_intent_status`, `_payment_method` (= `woocommerce_payments`), `_payment_method_title`, order status |
| **Order note patterns** | "Stripe payment intent ID:", "Payment via WooPayments", "WooPayments: Payment declined", "WooPayments: Charge captured", "Stripe Webhook:" |
| **Common failure modes** | 1. Test/live mode mismatch (test key on live store). 2. 3DS authentication cancelled by customer. 3. Webhook signing secret rotated without updating plugin. 4. WooPayments account restricted or under review. 5. APM gateway returning empty ID in unsupported country. |
| **Safe evidence fields** | `_wcpay_payment_intent_status` (string, no PII); payment method slug; order status; order note pattern counts; gateway enabled status; plugin version |
| **Official sources** | wordpress.org/plugins/woocommerce-payments; github.com/Automattic/woocommerce-payments |

---

### 2.2 WooCommerce Stripe (Official)

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-gateway-stripe` |
| **Gateway IDs** | `stripe` (primary card); `stripe_sepa`, `stripe_ideal`, `stripe_bancontact`, `stripe_giropay`, `stripe_p24`, `stripe_alipay`, `stripe_klarna`, `stripe_affirm`, `stripe_afterpay_clearpay` |
| **Log sources** | WC logger channel: `woocommerce-gateway-stripe` |
| **Webhook indicators** | `_stripe_source_id`, `_stripe_customer_id`, `_stripe_charge_id`, `_stripe_intent_id` in order meta; `_stripe_charge_captured` (bool); webhook events logged in WC logger |
| **Action Scheduler hooks** | `wc_stripe_...` hooks [verify specific names — Stripe plugin uses AS for some async capture flows] |
| **Safe order meta fields** | `_stripe_charge_captured` (boolean), `_payment_method`, `_payment_method_title`, order status |
| **Order note patterns** | "Stripe charge complete (Charge ID:", "Stripe payment intent created", "Payment via Stripe", "Stripe Webhook:", "Card declined", "Stripe Error:" |
| **Common failure modes** | 1. Webhook secret not configured or rotated. 2. Test/live key mismatch. 3. 3DS/SCA required but not handled. 4. Firewall blocking outbound to api.stripe.com. 5. UPE (Payment Element) enabled but blocks theme incompatible. |
| **Safe evidence fields** | `_stripe_charge_captured` (boolean); order status; payment method title; plugin version; checkout_type (block/shortcode — affects UPE availability); order note pattern counts |
| **Official sources** | wordpress.org/plugins/woocommerce-gateway-stripe; github.com/woocommerce/woocommerce-gateway-stripe |

---

### 2.3 WooCommerce PayPal Payments

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-paypal-payments` |
| **Gateway IDs** | `ppcp-gateway` (PayPal PPCP); `ppcp-credit-card-gateway` (Advanced Card Processing) |
| **Log sources** | WC logger channel: `woocommerce-paypal-payments` |
| **Webhook indicators** | `_ppcp_paypal_order_id` [verify], `_ppcp_paypal_intent` [verify], PayPal order status stored in order meta |
| **Action Scheduler hooks** | `woocommerce_paypal_payments_...` [verify — PPCP uses AS for subscription renewals and capture async flows] |
| **Safe order meta fields** | `_payment_method` (= `ppcp-gateway`), `_payment_method_title`, order status |
| **Order note patterns** | "PayPal PPCP:", "Capture completed", "PayPal payment captured", "Authorization voided", "PayPal Webhook received" |
| **Common failure modes** | 1. Sandbox/live switch without re-onboarding. 2. Webhook registration failing (PayPal requires HTTPS + public URL). 3. Orders stuck in `on-hold` when authorization not captured. 4. PDT/IPN conflict with legacy PayPal Standard (if both active). 5. CSP headers blocking PayPal JS SDK. |
| **Safe evidence fields** | Order status; payment method title; plugin version; webhook-related note pattern counts; whether `ppcp-gateway` appears in stuck order gateway summary |
| **Official sources** | wordpress.org/plugins/woocommerce-paypal-payments; github.com/woocommerce/woocommerce-paypal-payments |

---

### 2.4 WooCommerce Square

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-square` |
| **Gateway IDs** | `square_credit_card`; `square_cash_app_pay` [verify availability] |
| **Log sources** | WC logger channel: `woocommerce-square` [verify exact channel name] |
| **Webhook indicators** | Square payment ID in order meta [verify key name]; order meta set by Square sync |
| **Action Scheduler hooks** | Square inventory sync hooks (product sync, order sync) — `wc_square_...` [verify] |
| **Safe order meta fields** | `_payment_method` (= `square_credit_card`), `_payment_method_title`, order status |
| **Order note patterns** | "Square payment", "Transaction approved", "Square Webhook:", "Square Error:" [verify exact strings] |
| **Common failure modes** | 1. Wrong Square location ID configured. 2. Sandbox vs production environment mismatch. 3. Square inventory sync interfering with WooCommerce stock (dual inventory management conflict). 4. Square account not fully activated. 5. Card on file expired for returning customers. |
| **Safe evidence fields** | Order status; payment method title; plugin version; whether Square appears in gateway failure summary; note pattern counts |
| **Official sources** | wordpress.org/plugins/woocommerce-square; github.com/woocommerce/woocommerce-square |

---

### 2.5 Authorize.Net (SkyVerge / WooCommerce)

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-gateway-authorize-net-cim` (legacy SkyVerge); newer version may be under WooCommerce/SkyVerge umbrella [verify current slug] |
| **Gateway IDs** | `authorize_net_cim_credit_card`; `authorize_net_cim_echeck` [verify] |
| **Log sources** | WC logger channel tied to plugin [verify channel name] |
| **Webhook indicators** | Authorize.Net does not use webhooks in the traditional sense — uses Silent Post URL / IPN-style; transaction ID stored in order meta [verify key name] |
| **Action Scheduler hooks** | SkyVerge capture hooks [verify] |
| **Safe order meta fields** | `_payment_method`, `_payment_method_title`, order status; transaction ID meta (non-sensitive identifier) |
| **Order note patterns** | "Authorize.Net charge complete", "Transaction ID:", "AVS Mismatch", "Card declined" [verify exact strings] |
| **Common failure modes** | 1. API Login ID / Transaction Key in test vs live mode mismatch. 2. AVS/CVV mismatch causing decline. 3. Silent Post URL not reachable from Authorize.Net servers. 4. CIM customer profile creation failing. 5. IP restriction on Authorize.Net merchant account blocking server IP. |
| **Safe evidence fields** | Order status; payment method title; plugin version; note pattern counts for declines/errors |
| **Official sources** | [verify current WooCommerce.com/SkyVerge product URL — do not guess] |

---

### 2.6 Braintree

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-gateway-paypal-powered-by-braintree` (PayPal-owned) or SkyVerge variant [verify current status — PayPal acquired SkyVerge] |
| **Gateway IDs** | `braintree_credit_card`; `braintree_paypal` [verify] |
| **Log sources** | WC logger channel [verify] |
| **Webhook indicators** | Braintree webhook notification received → order meta or note [verify key] |
| **Action Scheduler hooks** | [verify — Braintree may use AS for subscription renewals via WC Subscriptions integration] |
| **Safe order meta fields** | `_payment_method`, `_payment_method_title`, order status; Braintree transaction ID (non-sensitive identifier) [verify key] |
| **Order note patterns** | "Braintree charge complete", "Transaction ID:", "Payment declined", "3D Secure authentication" [verify] |
| **Common failure modes** | 1. 3DS2 misconfiguration (required but not set up). 2. Merchant account ID mismatch (multiple merchant accounts). 3. Sandbox/production environment mismatch. 4. Vault payment method expired. 5. PayPal via Braintree conflicting with WC PayPal Payments plugin (two PayPal gateways). |
| **Safe evidence fields** | Order status; payment method title; plugin version; note pattern counts |
| **Official sources** | [verify current URL — Braintree for WooCommerce moved under PayPal ownership] |

---

### 2.7 Payment Plugins for Stripe (by Payment Plugins)

| Field | Value |
|---|---|
| **Plugin slug** | `woo-stripe-payment` [verify — Payment Plugins uses this slug on WP.org] |
| **Gateway IDs** | `stripe_payment_element`, `stripe_googlepay`, `stripe_applepay`, `stripe_klarna`, `stripe_affirm`, `stripe_afterpay_clearpay`, `stripe_sepa`, `stripe_ideal` [verify full list — Payment Plugins uses `stripe_` prefix for most sub-gateways] |
| **Log sources** | WC logger channel: `woo-stripe-payment` [verify] |
| **Webhook indicators** | Similar to official Stripe plugin — Stripe event IDs stored in order meta [verify key names differ from official plugin] |
| **Action Scheduler hooks** | [verify — Payment Plugins may use AS for some async flows] |
| **Safe order meta fields** | `_payment_method`, `_payment_method_title`, order status; Stripe intent status [verify key name vs official plugin] |
| **Order note patterns** | "Stripe payment complete", "Payment intent:", "Payment declined", "Stripe Error:" [verify — may differ from official plugin] |
| **Common failure modes** | 1. Conflict with official WooCommerce Stripe plugin (both active = duplicate gateway IDs). 2. Apple Pay domain verification not completed. 3. Google Pay merchant identifier not registered. 4. Webhook secret from wrong Stripe account. 5. Plugin-specific Stripe settings not migrated after switch from official plugin. |
| **Safe evidence fields** | Order status; payment method title; plugin version; whether `stripe_` prefix gateways appear in failure summary; note pattern counts |
| **Official sources** | wordpress.org/plugins/woo-stripe-payment |

---

### 2.8 Payment Plugins for PayPal (by Payment Plugins)

| Field | Value |
|---|---|
| **Plugin slug** | `pymntpl-paypal-woocommerce` [verify] |
| **Gateway IDs** | `ppcp`, `paypal_googlepay`, `paypal_applepay`, `paypal_venmo` [verify — Payment Plugins uses different gateway IDs from WooCommerce PayPal Payments] |
| **Log sources** | WC logger channel [verify] |
| **Webhook indicators** | PayPal order ID stored in order meta [verify key — likely different from official WooCommerce PayPal Payments] |
| **Action Scheduler hooks** | [verify] |
| **Safe order meta fields** | `_payment_method`, `_payment_method_title`, order status |
| **Order note patterns** | [verify — will differ from official PayPal plugin] |
| **Common failure modes** | 1. Conflict with WooCommerce PayPal Payments plugin (both active). 2. PayPal onboarding incomplete. 3. Webhook not registered. 4. Venmo or Apple Pay not enabled on PayPal account. |
| **Safe evidence fields** | Order status; payment method title; plugin version; whether Payment Plugins gateway IDs appear in failure summary |
| **Official sources** | wordpress.org/plugins/pymntpl-paypal-woocommerce [verify slug] |

---

### 2.9 Amazon Pay

| Field | Value |
|---|---|
| **Plugin slug** | `woocommerce-gateway-amazon-payments-advanced` |
| **Gateway IDs** | `amazon_payments_advanced` |
| **Log sources** | WC logger channel: `amazon_payments_advanced` |
| **Webhook indicators** | Amazon Reference ID / Order Reference ID stored in order meta [verify key name]; IPN notifications from Amazon Pay |
| **Action Scheduler hooks** | [verify — Amazon Pay may use AS for async capture or authorization renewal] |
| **Safe order meta fields** | `_payment_method` (= `amazon_payments_advanced`), `_payment_method_title`, order status |
| **Order note patterns** | "Amazon Pay:", "Authorization ID:", "Payment captured via Amazon Pay", "Amazon Pay declined" [verify] |
| **Common failure modes** | 1. CSP (Content-Security-Policy) header blocking Amazon Pay widget JS (common after security plugin install). 2. IPN endpoint not publicly reachable. 3. Sandbox vs production merchant account mismatch. 4. Amazon Pay account region mismatch (US vs EU). 5. Amazon Pay authorization expiring before capture (multi-day order processing). |
| **Safe evidence fields** | Order status; payment method title; plugin version; CSP header check [via checkout probe headers]; note pattern counts |
| **Official sources** | wordpress.org/plugins/woocommerce-gateway-amazon-payments-advanced; github.com/woocommerce/woocommerce-gateway-amazon-payments-advanced |

---

### 2.10 BNPL Gateways — Klarna, Affirm, Afterpay/Clearpay

BNPL gateways share common patterns. Key distinction: they are often **sub-gateways registered by one of the above plugins** (Stripe, Payment Plugins, or standalone).

| Gateway | Typical Plugin Source | Gateway ID(s) | Notes |
|---|---|---|---|
| Klarna | WC Stripe (`stripe_klarna`), WooPayments APM, or standalone Klarna plugin | `stripe_klarna`, `woocommerce_payments_klarna`, `klarna_payments` [verify standalone slug] | Availability restricted by country and currency |
| Affirm | WC Stripe (`stripe_affirm`), WooPayments APM | `stripe_affirm`, `woocommerce_payments_affirm` | US-only; order minimum required |
| Afterpay/Clearpay | WC Stripe (`stripe_afterpay_clearpay`), WooPayments APM, standalone Afterpay plugin | `stripe_afterpay_clearpay`, `woocommerce_payments_afterpay_clearpay`, `afterpay` | Name depends on region (Afterpay = US/AU, Clearpay = UK/EU) |

**Common BNPL failure modes:**
1. Order placed → customer redirected to BNPL portal → customer cancels → order stuck in `pending payment` (most common).
2. BNPL gateway available but order total below/above allowed range.
3. Country mismatch (Affirm shown to non-US customer).
4. Currency not supported.
5. BNPL plugin activated but API credentials not configured (gateway shows in list but returns empty ID or fails silently at payment step).

**Safe evidence fields for BNPL:** Order status; payment method title and ID; order total range (masked — bucket only: <$50, $50–$200, $200–$500, $500+); note pattern counts for decline/cancel events; country code from billing (only ISO code, not full address).

**Detection note:** If `pickup_location` or any non-payment method appears in the payment gateway list — as seen in the Samurai incident — it typically means a plugin or theme is incorrectly registering non-payment options via `woocommerce_payment_gateways` filter, or a BNPL/alternative method plugin has a registration bug.

---

## 3. Checkout Context Detection Plan

### A. Checkout Type Detection

**A1. Classic WooCommerce shortcode checkout**

- PHP: `has_shortcode(get_post(wc_get_page_id('checkout'))->post_content, 'woocommerce_checkout')` → returns `true`
- Implemented in `WRNSM_Checkout_Tracker::detect_checkout_type()` (v1.16.3)
- Additional signal: `function_exists('wc_deprecated_function')` does not distinguish; rely on post content check
- Fallback: If neither block nor shortcode detected and checkout page exists = `'unknown'`

**A2. WooCommerce Blocks checkout**

- PHP: `has_block('woocommerce/checkout', get_post(wc_get_page_id('checkout')))` → `true` if Gutenberg block registered
- Requires WordPress 5.0+ (`has_block()` available since WP 5.0)
- Implemented in `WRNSM_Checkout_Tracker::detect_checkout_type()` (v1.16.3)
- JS signal (supplemental): `typeof window.wc !== 'undefined' && typeof window.wc.blocksCheckout !== 'undefined'` detects blocks runtime
- Store API signal: checkout POST goes to `/wp-json/wc/store/v1/checkout` instead of `/?wc-ajax=checkout`

**A3. Store API checkout path**

- Detection: Whether `/wp-json/wc/store/v1/checkout` endpoint returns 200 or 404
- Probe approach: HTTP HEAD to `home_url('/wp-json/wc/store/v1/checkout')` — a 404 means Store API not available (WC Blocks not installed/active); a 401 (unauthorized) or 400 (bad request) means available but requires auth — expected
- Note: Block checkout always uses Store API. Classic shortcode checkout uses `wc-ajax=checkout`. A store can have both block and shortcode on different pages — detection must check the actual checkout page content, not just plugin presence.

**A4. Page builder checkout**

| Page Builder | Detection Signal |
|---|---|
| Elementor | Post content contains `[elementor-template` shortcode or `elementor` CSS class in checkout page post meta; `is_plugin_active('elementor/elementor.php')` |
| Divi | Post content uses Divi shortcode format `[et_pb_section...]`; `ET_CORE_VERSION` constant defined |
| Beaver Builder | Post content contains `[fl_builder_insert_layout`; `FL_BUILDER_VERSION` constant defined |
| WPBakery | Post content contains `[vc_row]`; `WPB_VC_VERSION` constant defined |

When a page builder is detected on the checkout page, note this as a `checkout_context_risk` because:
- Page builders can inject scripts in non-standard order
- Theme-specific JS execution order may conflict with WooCommerce checkout JS
- Some page builders render shortcodes in ways that break WooCommerce's form field scanning

---

### B. Gateway Script Loading

**B1. Detect which gateway scripts are enqueued on checkout**

- During checkout page render: hook `wp_enqueue_scripts` at priority 999 and inspect `$wp_scripts->queue` for gateway-related handles
- Store the list in a transient (`wrnsm_checkout_scripts_snapshot`) — refresh on each checkout page probe
- Key handles to look for:
  - Stripe: `stripe`, `wc-stripe`, `wc-gateway-stripe`, `woocommerce-gateway-stripe`
  - WooPayments: `wcpay-checkout`, `wcpay-payment-request`
  - PayPal: `ppcp-smart-button`, `woocommerce-paypal-payments`
  - Square: `square-js`, `wc-square`
  - Amazon Pay: `amazon-pay-checkout`

**B2. Detect script loading order conflicts**

- A conflict exists when a gateway requires jQuery but jQuery is enqueued in the footer (low priority) while the gateway script is in header
- Check: `$wp_scripts->registered['stripe']->deps` contains `'jquery'`; check `$wp_scripts->in_footer` for `'jquery'`
- Also: detect if any caching/optimization plugin (WP Rocket, LiteSpeed Cache, W3 Total Cache, Autoptimize) is active — these can defer or combine scripts in ways that break checkout gateway JS
- Known conflict pattern: script defer/async on gateway JS = breaks Stripe Elements initialization

**B3. Detect missing gateway scripts**

- If a gateway is marked enabled in WC settings but its expected JS handle is not in `$wp_scripts->queue` on the checkout page = potential issue
- Build a map of gateway ID → expected JS handle for known gateways
- Requires the detection to run on an actual frontend request (not admin), so this is best done during a checkout page HTTP probe with a full WP bootstrap [limitation: current probe is HEAD-only]

---

### C. Payment Method Registration

**C1. Detect gateways returning invalid IDs**

```php
// Safe detection approach:
$gateways = WC()->payment_gateways()->payment_gateways();
$available = WC()->payment_gateways()->get_available_payment_gateways();
foreach ($gateways as $id => $gateway) {
    if ('' === $id || ! is_string($id)) {
        // Invalid gateway ID — registration bug
    }
    if ('yes' === $gateway->enabled && ! isset($available[$id])) {
        // Enabled but not available — filter removing it
    }
}
```

**C2. Detect conditional availability issues**

- Some gateways use `is_available()` to exclude themselves based on: currency, country, order total, product type, shipping method
- Detection: call `is_available()` on each enabled gateway; compare registered list vs returned list; log the difference as `[gateway_id: registered but unavailable]`
- This is what exposes the "no payment methods" state without triggering it on real customers

**C3. Detect pickup/shipping methods leaking into payment context (Samurai pattern)**

- Root cause: a plugin incorrectly registers a non-payment method via `woocommerce_payment_gateways` filter instead of `woocommerce_shipping_methods`
- Detection signals:
  - Gateway ID contains terms like `pickup`, `local`, `shipping`, `delivery`, `store`, `collect`
  - Gateway plugin class name extends a shipping base class instead of `WC_Payment_Gateway`
  - `WC()->payment_gateways()->payment_gateways()` returns an entry where `instanceof WC_Payment_Gateway` is false or where `$gateway->get_order_button_text()` returns a shipping-specific string
- Specific check: `isset($gateways['pickup_location'])` or similar pattern — log as `checkout_context_risk: non_payment_method_in_gateway_list`

---

### D. Checkout Page Health Signals

| Signal | How to collect | Healthy result | Problem result |
|---|---|---|---|
| Checkout page HTTP status | HEAD probe to `wc_get_checkout_url()` | 200 | 404, 301/302 (redirect loop), 500 |
| Cart page HTTP status | HEAD probe to `wc_get_cart_url()` | 200 | 404, 500 |
| Order-received endpoint | HEAD probe to `wc_get_endpoint_url('order-received', '0', wc_get_checkout_url())` | 200 | 404, 500 |
| WC-AJAX checkout endpoint | POST probe to `admin-ajax.php?action=woocommerce_checkout` with minimal nonce | Returns WC checkout validation response | Connection refused, timeout, 500 |
| Store API availability | HEAD probe to `rest_url('wc/store/v1/cart')` | 200 or 401 | 404 (blocks not installed), 500 |
| Response time | Time HEAD request to checkout URL | < 1500ms | > 3000ms = degraded, > 5000ms = critical |

All probe results should be cached in transients with 5–15 min TTL to avoid over-probing. Current implementation already caches order-received and checkout probes.

---

## 4. Controlled Evidence Tools Plan

All tools live in a new class: `WRNSM_Evidence_Tools`. No arbitrary SQL. All queries use WP/WC APIs or prepared statements via `$wpdb->prepare()` on controlled table/column names. No tool returns PII.

---

### Tool 1: `get_checkout_type_context()`

**Method signature:**
```php
public static function get_checkout_type_context(): array
```

**What it queries:**
- `WRNSM_Checkout_Tracker::detect_checkout_type()` for checkout_type
- `WC()->payment_gateways()->payment_gateways()` for registered gateways
- `WC()->payment_gateways()->get_available_payment_gateways()` for available gateways
- `wc_get_page_id('checkout')` and `get_post()` for page builder detection
- Transient `wrnsm_checkout_scripts_snapshot` for enqueued scripts if available

**Output structure:**
```php
[
    'checkout_type'          => 'block' | 'shortcode' | 'unknown',
    'page_builder_detected'  => 'none' | 'elementor' | 'divi' | 'beaver_builder' | 'wpbakery',
    'registered_gateways'    => ['stripe', 'paypal', ...],       // array of IDs only
    'available_gateways'     => ['stripe', ...],                 // IDs of actually-available gateways
    'unavailable_gateways'   => ['paypal', ...],                 // enabled but filtered out
    'invalid_gateway_ids'    => [...],                           // empty-string or non-string IDs
    'non_payment_in_list'    => ['pickup_location0', ...],       // suspicious IDs
    'store_api_reachable'    => true | false | null,             // null = not probed
    'checkout_scripts'       => ['stripe', 'wcpay-checkout'],    // known handles if snapshot available
]
```

**Safety rules:** Never return API keys, secret keys, or gateway credentials. Strip any gateway option whose key name contains `key`, `secret`, `token`, `password`, or `credential`.

**AI usage guidance:** Use `checkout_type` to constrain jQuery vs MutationObserver event interpretation. Use `non_payment_in_list` as primary signal for the "invalid payment method" issue class. Use `unavailable_gateways` to identify filtered-out gateways. If `available_gateways` is empty = customers cannot pay.

---

### Tool 2: `get_recent_checkout_error_clusters()`

**Method signature:**
```php
public static function get_recent_checkout_error_clusters( int $days = 7, int $limit = 10 ): array
```

**What it queries:**
- `wrnsm_events` table via `$wpdb->prepare()`: `WHERE event_type = %s AND created_at > %s`
- Groups by normalized message text (strip order IDs, amounts, product names)
- Max 100 raw events fetched, aggregated in PHP

**Output structure:**
```php
[
    [
        'message'          => 'Payment declined — insufficient funds',  // normalized, 150 char cap
        'count'            => 12,
        'first_seen'       => '2026-06-28 14:22:00',
        'last_seen'        => '2026-06-30 09:15:00',
        'gateway_slugs'    => ['stripe'],                              // gateway IDs seen with this error
        'is_accelerating'  => true,                                    // count doubled in last 24h vs prior 24h
    ],
    ...
]
```

**Safety rules:** Normalize messages before returning: replace numbers with `{N}`, email patterns with `{email}`, URLs with `{url}`. Cap message at 150 chars. Never return raw event row data.

**AI usage guidance:** Cluster size and velocity are primary signals. If `is_accelerating` = true and a cluster matches a gateway decline pattern = active incident. If multiple clusters with different gateways = store-wide issue. Compare `first_seen` against plugin lifecycle events for correlation.

---

### Tool 3: `get_invalid_gateway_id_summary()`

**Method signature:**
```php
public static function get_invalid_gateway_id_summary(): array
```

**What it queries:**
- `WC()->payment_gateways()->payment_gateways()` — all registered
- `WC()->payment_gateways()->get_available_payment_gateways()` — available on checkout
- Inspects each gateway for: empty ID, non-string ID, ID matching pickup/shipping patterns, `instanceof WC_Payment_Gateway` check

**Output structure:**
```php
[
    'registered_count'   => 5,
    'available_count'    => 3,
    'invalid_ids'        => [],                                   // empty string IDs
    'suspicious_ids'     => ['pickup_location0'],                 // pickup/shipping-pattern IDs
    'filtered_out'       => ['paypal'],                           // enabled but not available
    'registration_issues'=> [
        ['id' => 'pickup_location0', 'reason' => 'matches_pickup_pattern', 'class' => 'WC_Local_Pickup_Plus_Shipping_Method'],
    ],
]
```

**Safety rules:** Return class names only, never class instances. Never return gateway settings arrays.

**AI usage guidance:** `suspicious_ids` non-empty = root cause of "no payment methods" or "invalid payment method" in block checkout. Match `registration_issues[*].class` against active plugin list to identify the offending plugin.

---

### Tool 4: `get_affected_orders_summary()`

**Method signature:**
```php
public static function get_affected_orders_summary( int $days = 7 ): array
```

**What it queries:**
- `wc_get_orders()` with HPOS-compatible args: date range, status groups
- Returns aggregate counts only — never individual order data

**Output structure:**
```php
[
    'date_range_days'     => 7,
    'status_counts'       => [
        'pending'         => 3,
        'failed'          => 12,
        'on-hold'         => 5,
        'processing'      => 45,
        'completed'       => 201,
        'cancelled'       => 8,
        'refunded'        => 2,
    ],
    'stuck_pending_over_1h'  => 3,    // pending and created > 1h ago
    'stuck_processing_over_24h' => 1, // processing and created > 24h ago (no shipment update)
    'gateway_distribution'   => [
        'stripe'          => 180,
        'paypal'          => 60,
        'woocommerce_payments' => 28,
    ],
    'payment_method_totals_scrubbed' => true,   // confirms amounts not included
]
```

**Safety rules:** No order IDs. No customer names/emails. No line items. No billing/shipping addresses. No amounts (aggregate counts only). No order notes.

**AI usage guidance:** `failed` count spike + specific gateway in `gateway_distribution` = gateway-specific issue. `stuck_pending_over_1h` high = webhook not delivering or WP-Cron not running. Compare gateway distribution against gateway failure summary to identify disproportionate failure rates.

---

### Tool 5: `get_gateway_failure_summary()`

**Method signature:**
```php
public static function get_gateway_failure_summary( int $days = 7 ): array
```

**What it queries:**
- `WRNSM_Payment_Health::get_gateway_stats($days)` for per-gateway success/failure rates
- `WRNSM_Payment_Health::get_summary()` for overall rate
- `wrnsm_events` for payment_failed events by gateway (last occurrence)

**Output structure:**
```php
[
    'overall_success_rate'   => 0.87,
    'overall_failure_count'  => 15,
    'days'                   => 7,
    'gateways'               => [
        [
            'gateway_id'      => 'stripe',
            'success_rate'    => 0.92,
            'total'           => 120,
            'failed'          => 10,
            'last_failure_at' => '2026-06-30 08:14:00',
            'trend'           => 'worsening' | 'stable' | 'improving' | 'insufficient_data',
        ],
        ...
    ],
    'worst_gateway'          => 'paypal',   // lowest success rate
    'is_multi_gateway_issue' => false,      // true if 2+ gateways showing elevated failure
]
```

**Safety rules:** Rates and counts only. No order IDs, amounts, card types, or customer data.

**AI usage guidance:** Single gateway failure rate drop = gateway-specific issue. Multi-gateway failure = store-wide (checkout page, SSL, caching, critical plugin). `last_failure_at` near a `gateway_change` event = possible settings change caused failures. Rates below 0.85 = significant signal. Below 0.70 = urgent.

---

### Tool 6: `get_action_scheduler_gateway_context()`

**Method signature:**
```php
public static function get_action_scheduler_gateway_context(): array
```

**What it queries:**
- `as_get_scheduled_actions()` with hook pattern filters for known gateway prefixes: `wcpay_`, `wc_stripe_`, `woocommerce_paypal_`, `wc_square_`, `amazon_pay_`
- Groups by hook name and status (pending/complete/failed/in-progress/canceled)

**Output structure:**
```php
[
    'as_active'        => true,
    'gateway_hooks'    => [
        [
            'hook'     => 'wcpay_track_new_invoice',
            'pending'  => 0,
            'complete' => 45,
            'failed'   => 2,
            'last_failed_at' => '2026-06-29 11:30:00',
        ],
        ...
    ],
    'total_failed'     => 2,
    'total_pending'    => 0,
    'oldest_pending_age_hours' => null,   // null if no pending jobs
]
```

**Safety rules:** Return hook names and counts only. Never return job arguments (may contain order IDs or tokens). Never return group names that contain customer identifiers.

**AI usage guidance:** Failed gateway-specific hooks + stuck orders on that gateway = webhook processing or async capture failure. Large `pending` count + `oldest_pending_age_hours` > 2 = AS queue backed up = cron issue. Combine with `get_gateway_failure_summary()` for full picture.

---

### Tool 7: `get_slow_request_summary()`

**Method signature:**
```php
public static function get_slow_request_summary(): array
```

**What it queries:**
- `WRNSM_Checkout_Prober::get_last()` for last checkout probe result
- Transients for cart probe and AJAX probe results
- Scan history (last 10 scans) for `checkout_response_ms` trend from `snapshot_data`

**Output structure:**
```php
[
    'checkout_last_ms'    => 2340,
    'checkout_status'     => 'ok' | 'slow' | 'failed' | 'never_run',
    'checkout_checked_at' => '2026-06-30 09:00:00',
    'cart_last_ms'        => 1800,
    'cart_status'         => 'ok',
    'ajax_last_ms'        => 450,
    'ajax_status'         => 'ok',
    'store_api_last_ms'   => null,   // not yet probed
    'trend_10_scans'      => [2100, 2200, 2300, 2340, ...],  // checkout response ms per scan
    'trend_direction'     => 'worsening' | 'stable' | 'improving' | 'insufficient_data',
    'thresholds'          => ['slow' => 2000, 'critical' => 4000],
]
```

**Safety rules:** Times and HTTP status codes only. No URL paths containing customer data. No server identifiers.

**AI usage guidance:** `trend_direction = worsening` over 10 scans = progressive performance issue. All endpoints slow = server-level issue. Only checkout slow = checkout-specific (large plugin stack on checkout, gateway JS loading slowly). `store_api_last_ms` missing = Store API probe not implemented yet (gap to flag).

---

### Tool 8: `get_plugins_touching_checkout_context()`

**Method signature:**
```php
public static function get_plugins_touching_checkout_context(): array
```

**What it queries:**
- `$wp_filter` global for specific hooks: `woocommerce_checkout_order_created`, `woocommerce_checkout_order_processed`, `woocommerce_payment_complete`, `woocommerce_available_payment_gateways`, `woocommerce_checkout_fields`, `woocommerce_payment_gateways`
- Maps callback file paths back to plugin slugs via `plugin_basename()`

**Output structure:**
```php
[
    'woocommerce_available_payment_gateways' => [
        'woocommerce-local-pickup-plus/woocommerce-local-pickup-plus.php',
        'woocommerce-subscriptions/woocommerce-subscriptions.php',
    ],
    'woocommerce_payment_gateways' => [
        'woocommerce-local-pickup-plus/woocommerce-local-pickup-plus.php',
    ],
    'woocommerce_checkout_fields' => [
        'woocommerce-checkout-field-editor/woocommerce-checkout-field-editor.php',
    ],
    'woocommerce_payment_complete' => [
        'woocommerce-subscriptions/woocommerce-subscriptions.php',
    ],
    'summary' => [
        'plugins_touching_payment_gateways' => [
            'woocommerce-local-pickup-plus',
        ],
        'plugins_touching_checkout_fields' => [
            'woocommerce-checkout-field-editor',
        ],
    ],
]
```

**Safety rules:** Return plugin file basenames only. Never return callback function names or closure code. Never return hook priority numbers (unnecessary for AI context).

**AI usage guidance:** If a plugin appears in `woocommerce_payment_gateways` that is not a payment gateway plugin by name = strong suspect for payment method list pollution (Samurai pattern). If multiple plugins are touching `woocommerce_available_payment_gateways` = conflict risk. Cross-reference with plugin lifecycle events — if one was recently activated = strong correlation.

---

### Tool 9: `get_order_notice_summary()`

**Method signature:**
```php
public static function get_order_notice_summary( int $days = 7, int $sample_size = 50 ): array
```

**What it queries:**
- `wc_get_orders()` limited to `$sample_size` most recent orders with `failed`, `on-hold`, or `pending` status
- For each order, `wc_get_order_notes($order_id)` — pattern-matches against known gateway note patterns
- Returns counts per pattern, never raw note text

**Known patterns to match:**
```
'stripe_charge_complete' => '/stripe charge complete/i',
'stripe_intent_created'  => '/stripe payment intent/i',
'stripe_declined'        => '/stripe.*declin/i',
'paypal_captured'        => '/paypal.*captur/i',
'paypal_declined'        => '/paypal.*declin/i',
'wcpay_intent'          => '/wcpay.*intent/i',
'webhook_received'       => '/webhook receiv/i',
'webhook_failed'         => '/webhook.*fail/i',
'avs_mismatch'           => '/avs.*mismatch/i',
'cvv_failed'             => '/cvv|cvc.*fail/i',
'authorization_void'     => '/authoriz.*void/i',
'capture_failed'         => '/capture.*fail/i',
'3ds_failed'             => '/3d.?s|3ds.*fail/i',
'refund_note'            => '/refund.*process/i',
```

**Output structure:**
```php
[
    'sample_size'        => 50,
    'date_range_days'    => 7,
    'pattern_counts'     => [
        'stripe_declined'   => 12,
        'webhook_received'  => 3,
        '3ds_failed'        => 4,
        'avs_mismatch'      => 2,
    ],
    'gateways_seen_in_notes' => ['stripe', 'woocommerce_payments'],
    'no_gateway_notes'       => 18,  // orders with no recognizable gateway note pattern
]
```

**Safety rules:** Never return raw order note text. Never return order IDs. Never return customer names. Pattern matching only in PHP; only counts returned.

**AI usage guidance:** High `stripe_declined` count = Stripe-side declines (may be bank-side — human gateway dashboard needed). High `3ds_failed` = 3DS misconfiguration or bank enforcement. High `no_gateway_notes` in failed orders = payment silently failed before reaching gateway = WooCommerce-layer or checkout-layer issue. `webhook_failed` pattern count > 0 = webhook processing issue.

---

## 5. AI UX Consolidation Plan

### Level 1 — Store Health Summary AI (existing: WRNSM_AI_Advisor, overview scope)

| Field | Detail |
|---|---|
| **Trigger** | Manual button in admin dashboard or after each scheduled scan |
| **Model** | `claude-haiku-4-5-20251001` |
| **Input** | Current scan result array (all monitors, severity scores, health score) |
| **Output** | 2–4 sentence plain-English summary of store health; highlights critical monitors; suggests next step |
| **What it can conclude** | Health is good/degraded/critical; which areas need attention; general trend |
| **What it cannot conclude** | Root cause; specific fix; gateway-level diagnosis |
| **Admin UX** | Collapsed card in dashboard overview; refresh button; timestamp |
| **Safe claim** | "Helps store owners understand their WooCommerce health at a glance" |

### Level 2 — Monitor AI (existing: WRNSM_AI_Advisor, per-monitor scope)

| Field | Detail |
|---|---|
| **Trigger** | Admin clicks "Explain this" on any monitor result |
| **Model** | `claude-haiku-4-5-20251001` |
| **Input** | Single monitor result (severity, summary, details, meta) + store context |
| **Output** | 2–4 sentence explanation of what this monitor found and what it likely means |
| **What it can conclude** | What the signal means in context; likely causes; what to check next |
| **What it cannot conclude** | Whether the problem is active; whether it caused a specific order failure; gateway-specific root cause |
| **Admin UX** | Inline expandable below each monitor card; "Ask Follow-up" conversation (existing) |
| **Safe claim** | "Explains monitor results in plain English" |

### Level 3 — Diagnosis AI (existing: WRNSM_AI_Diagnosis, incident scope)

| Field | Detail |
|---|---|
| **Trigger** | Manual — admin fills in incident form, clicks "Generate Diagnosis" |
| **Model** | `claude-sonnet-4-6` |
| **Input** | Full diagnostic payload (12+ sections, up to 12,000 chars): incident context, scan snapshot, event aggregates, payment health stats, checkout probe, top checkout errors, missing data, active plugins, environment, order status summary |
| **Output** | 13-key JSON schema: symptom_pattern, primary_hypothesis, supporting_hypotheses, suggested_only, confidence, confidence_readiness, missing_evidence, priority, urgency, recommended_action, ruled_out, caveats, checkout_context_note |
| **What it can conclude** | Ranked hypothesis with supporting evidence; what's missing; urgency level |
| **What it cannot conclude** | Definitive root cause without gateway dashboard access; whether fix worked; specific Stripe/PayPal error code without decline code |
| **Admin UX** | Diagnosis tab — 3-step wizard: (1) fill incident form, (2) generate payload, (3) run diagnosis; result rendered with formatted sections; confidence badge |
| **Safe claim** | "AI-assisted triage using available store data; human verification required for confirmed diagnosis" |

### Level 4 — Evidence Tools (proposed)

**Architecture:** Diagnosis AI in Level 3 produces its initial diagnosis from the static payload. If confidence is `limited` or `partial`, the UI offers "Deepen Evidence" — which calls Level 4 tools and appends results to a second AI pass.

**Mechanism:**
1. `WRNSM_AI_Diagnosis::generate()` produces initial diagnosis
2. PHP reads `confidence_readiness` from JSON result
3. If `limited` or `partial`: UI shows "Gather additional evidence" button
4. Admin clicks → PHP calls `WRNSM_Evidence_Tools` methods relevant to the symptom category
5. Results appended to the prompt as a new `EVIDENCE TOOLS` section
6. Second Sonnet pass with the enriched context
7. New diagnosis stored with `trigger_type = 'evidence_enriched'`

**Tool result format in AI context:**
```
EVIDENCE TOOLS (gathered on request):

[get_checkout_type_context]
checkout_type: block
available_gateways: stripe
non_payment_in_list: pickup_location0
suspicious_ids: true

[get_recent_checkout_error_clusters]
"no payment methods available" — count: 82, accelerating: true, gateways: none
"payment declined" — count: 3, accelerating: false, gateways: stripe

[get_invalid_gateway_id_summary]
suspicious: pickup_location0 (class: WC_Local_Pickup_Plus_Shipping_Method)
registered_by_hook: woocommerce_payment_gateways

[get_plugins_touching_checkout_context]
woocommerce_payment_gateways: woocommerce-local-pickup-plus
```

**Tool result trust levels:**
- `get_invalid_gateway_id_summary` + `get_checkout_type_context` + `get_plugins_touching_checkout_context` → justify strong hypothesis (can name the plugin as prime suspect)
- `get_gateway_failure_summary` + `get_order_notice_summary` → justify medium-confidence gateway hypothesis
- `get_affected_orders_summary` alone → informational only (confirms scale, not cause)
- `get_slow_request_summary` → justify performance hypothesis only if `trend_direction = worsening`

### Level 5 — Known-Issue Research (proposed)

**Mechanism:** Compare installed plugin versions against WP.org API for latest version. For gateways, pattern-match installed version against a local known-issues registry.

```php
// Known-issues registry structure (static PHP array, updated manually or via WRN KB sync):
[
    'woocommerce-gateway-stripe' => [
        '8.6.0' => 'Block checkout payment request button conflicts with Blocks API in WC 9.x',
        '8.7.1' => 'Fixed: Webhook processing race condition under high load',
    ],
    'woocommerce-payments' => [
        '7.8.0' => 'WooPayments APM gateways may return empty ID in non-US stores',
    ],
]
```

**Safety rule:** Never hallucinate known issues. The registry is explicitly maintained. If an installed version is not in the registry, return `no_known_issues: true`. AI should cite known issues as "a known issue was recorded for this version" — not as confirmed fact.

**Admin UX:** In Diagnosis tab, below the AI result: "Plugin version check" expandable showing: installed version, latest version, whether update available, whether current version has a known recorded issue.

---

### AI UX Flow Summary

```
Admin notices a problem
         ↓
Dashboard Overview (Level 1 AI)
"Your store health is 62 — payment failures elevated on Stripe"
         ↓
Admin clicks Stripe monitor → Monitor AI (Level 2)
"15 payment failures detected in 7 days. Stripe success rate dropped from 94% to 79% on June 28."
         ↓
Admin goes to Diagnosis tab, fills incident form, clicks Generate (Level 3 AI)
"Primary hypothesis: webhook processing failure — gateway_change event on June 27 correlates with failure onset. Confidence: partial. Missing: Stripe webhook delivery logs."
         ↓
Admin clicks "Gather additional evidence" (Level 4)
Evidence tools run → second AI pass
"Confirmed: woocommerce_payment_gateways filter is registered by woocommerce-local-pickup-plus. This plugin appears to inject a non-payment shipping method into the payment list. High confidence."
         ↓
Admin shares diagnosis + evidence with WebReadyNow (human layer)
WebReadyNow verifies, deactivates plugin on staging, tests, applies fix on production.
```

---

## 6. Recommended Next 3 Sprints

### Sprint A — v1.17.0: Controlled Evidence Layer

**Goal:** Build the `WRNSM_Evidence_Tools` class with all 9 controlled tools, integrate it into the diagnostic payload and Diagnosis tab UI, and build the checkout context scanner.

**What to build:**

| Component | File | Description |
|---|---|---|
| `WRNSM_Evidence_Tools` | `includes/class-wrnsm-evidence-tools.php` | All 9 tools as public static methods |
| `WRNSM_Gateway_Registry` | `includes/class-wrnsm-gateway-registry.php` | Known gateway ID → metadata map (plugin slug, safe meta keys, note patterns); static array, no external calls |
| Checkout context scanner | Extension of `WRNSM_Monitor_Checkout` | Add: invalid gateway ID check, non-payment method detection, filter hook inspection |
| Payload integration | `class-wrnsm-diagnostic-payload.php` | Add `evidence_tools_summary` section built from tool results (on-demand only, not in every payload) |
| Diagnosis tab UX | `admin/views/tabs/tab-diagnosis.php` | After initial diagnosis, show "Confidence: limited → Gather additional evidence" button |
| Evidence enrichment | `class-wrnsm-ai-diagnosis.php` | Add `generate_enriched($payload_id, $tool_results)` method; second Sonnet pass |
| Plugin hooks inspector | In `WRNSM_Evidence_Tools` | `get_plugins_touching_checkout_context()` using `$wp_filter` |

**Version bump:** 1.16.3 → 1.17.0

**What becomes safely claimable:**
- "Detects non-payment methods injected into the WooCommerce payment gateway list"
- "Identifies which active plugins are filtering or modifying payment method availability"
- "Provides structured gateway failure rates per gateway for AI-assisted triage"
- "Gathers additional targeted evidence when initial diagnosis confidence is low"

**What should still not be claimed:**
- Root cause confirmation without human gateway dashboard verification
- Automatic fix of any identified issue
- Complete detection of all checkout failures

---

### Sprint B — v1.18.0: Evidence UX + Guided Investigation

**Goal:** Surface evidence tool results to the admin in a readable, actionable panel, and introduce a guided incident investigation flow that replaces the current flat incident form.

**What to build:**

| Component | File | Description |
|---|---|---|
| Checkout Context panel | New section in `tab-diagnosis.php` or new tab | Shows checkout_type, available gateways, suspicious IDs, filter hooks — always visible, refreshes on demand |
| Gateway Health panel | Expand existing payment health section | Per-gateway success rate chart (last 30 days); failure trend indicator; last failure timestamp |
| Evidence timeline correlation | `tab-activity.php` extension | Show plugin lifecycle events and gateway change events on the same timeline as checkout errors; highlight correlations |
| Plugin conflict indicator | `tab-diagnosis.php` | If a plugin appears in `woocommerce_payment_gateways` and is not a known payment gateway = red flag card |
| Guided incident flow | `tab-diagnosis.php` | Replace flat form with a 3-step wizard: (1) describe symptom, (2) system auto-selects relevant evidence tools, (3) AI diagnosis with enrichment |
| Version check panel | `tab-diagnosis.php` | Show installed vs latest version for key gateway plugins; surface known-issue registry hits |
| Store API probe | `WRNSM_Checkout_Prober` extension | Add Store API endpoint probe (`/wp-json/wc/store/v1/cart`) |

**Version bump:** 1.17.x → 1.18.0

**What becomes safely claimable:**
- "Shows which plugins are modifying payment gateway availability on checkout"
- "Surfaces checkout context signals — checkout type, available gateways, script loading — in the admin"
- "Correlates plugin activations and gateway setting changes with checkout error patterns on a shared timeline"
- "Compares installed gateway plugin versions with latest available versions"

**What should still not be claimed:**
- Confirmed root cause without human verification
- Gateway dashboard access or webhook delivery logs
- Complete conflict detection (plugin conflicts not involving payment hooks are out of scope)

---

### Sprint C — v1.19.0: Managed Service Handoff

**Goal:** Build the structured pathway from plugin-detected evidence to WebReadyNow human service engagement, including a client-facing evidence summary and priority escalation mechanism.

**What to build:**

| Component | File | Description |
|---|---|---|
| Evidence summary export | `WRNSM_Report_Exporter` extension | Redacted client-facing PDF/HTML summary: health score, incident context, AI hypothesis, evidence tool summary — no raw events, no PII, no API keys |
| Escalation panel | New section in `tab-diagnosis.php` | Appears when confidence = limited OR urgency = high/critical; shows "Get WebReadyNow help" CTA with pre-filled context |
| Priority routing v2 | `WRNSM_Alert_Manager` extension | Route high-confidence diagnosis directly to priority alert channel; include AI hypothesis in alert body |
| Managed service status | New `tab-settings.php` section | Whether store is under WebReadyNow managed monitoring; service tier; last WebReadyNow review date |
| WRN Proxy AI | Replace direct Anthropic key | Route AI calls through WRN Hub proxy for usage caps per license tier (pre-release blocker) |
| License gate | `WRNSM_License` implementation | Call `is_valid()` before AI features; show upgrade notice if unlicensed (pre-release blocker) |

**Version bump:** 1.18.x → 1.19.0

**What becomes safely claimable:**
- "Helps store owners share structured WooCommerce diagnostic evidence with their support team"
- "Provides a direct pathway to WebReadyNow managed monitoring when human diagnosis is needed"
- "Prioritized escalation for high-urgency incidents with AI-assisted context"

**What should still not be claimed:**
- Guaranteed response time without a defined SLA
- Automatic escalation or ticket creation (requires human approval)
- Managed monitoring without an explicit subscription agreement

---

### What to Explicitly Postpone (Not in Next 3 Sprints)

| Feature | Why Postponed |
|---|---|
| Synthetic checkout testing (cart/payment simulation) | Requires headless browser or server-side puppeteer setup; high complexity; risk of triggering real payment flows |
| Gateway dashboard integration (Stripe/PayPal API direct calls) | Requires storing gateway API credentials securely; out of scope for a local WordPress plugin; belongs in a managed service layer with explicit consent |
| Real-time alerting via gateway webhooks | Would require the plugin to act as a webhook receiver and process gateway events — much higher attack surface than current approach |
| Email delivery tracking via SMTP API integration | Plugin-specific (FluentSMTP, WP Mail SMTP, Postmark, SendGrid all have different APIs); scope creep; best handled by those plugins |
| Multi-site / network admin support | Adds significant complexity; defer until single-site is stable |
| Public-facing status page | Requires external hosting/CDN; out of scope for a local plugin |
| Automatic remediation of any kind | Explicitly prohibited by product direction and safe-claims rules |

---

## Safety Constraints — What WRN Store Monitor Will Never Do

1. **Never automatically fix anything.** The plugin detects, logs, summarizes, and alerts. Fixes require human action. Code that calls `update_option()`, `wp_delete_post()`, modifies plugin settings, or sends data to a gateway on behalf of the store is never acceptable.

2. **Never claim complete coverage.** The plugin surfaces observable WooCommerce-layer signals. It cannot observe server-side failures, gateway dashboard events, DNS issues, bank-side declines, or infrastructure problems.

3. **Never run arbitrary SQL.** All database access uses WP/WC APIs or `$wpdb->prepare()` with controlled query patterns. No `eval()`, no dynamic table names, no AI-generated SQL execution.

4. **Never collect PII.** No customer names, emails, phone numbers, shipping addresses, IP addresses, card data, order notes containing customer communication, or any personally identifiable information leaves the controlled evidence tool output.

5. **Never collect secrets.** No API keys, webhook signing secrets, private keys, passwords, OAuth tokens, or credentials of any kind. Evidence tools explicitly strip any value whose key name pattern matches `key`, `secret`, `token`, `password`, `credential`, `auth`.

6. **Never replace human WooCommerce support.** The AI provides hypotheses and organizes evidence. Root-cause confirmation, fix implementation, staging testing, gateway dashboard verification, and client communication are always human responsibilities.

7. **Never guarantee alert delivery.** Alerts are logged with status. Inbox placement, Slack delivery, and notification reception depend on external systems outside plugin control.

8. **Never imply AI diagnosis is confirmed.** All AI output is labeled as hypothesis or triage. Every diagnosis includes a `caveats` field and a `missing_evidence` field explaining what would be needed for higher confidence.

9. **Never send store data to AI without explicit admin action.** No automatic AI submission. Every AI call is triggered by an admin button click. Data sent is shown to the admin before submission (payload preview).

10. **Never simulate payment submission.** No cart filling, no payment form submission, no headless browser interaction with the checkout flow.

---

*Document end. Next review: after Sprint A (v1.17.0) ship.*
