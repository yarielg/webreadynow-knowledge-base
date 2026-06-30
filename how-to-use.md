# WebReadyNow AI Framework — Session Manual

**Purpose:** Copy and paste from this file. Use it every session so you always run the framework correctly and get useful outputs.

---

## The 6 Agents and What They Do

| Agent | Prompt File | Use It When |
|---|---|---|
| **Researcher Agent** | `agents/researcher-agent.md` | You need to validate an idea, understand buyer pain, research competitors, or check if an opportunity is real before committing to it |
| **Product & Offer Alignment Agent** | `agents/product-offer-alignment-agent.md` | You have a validated idea and need to decide how to package it — plugin, service, productized service, plugin + setup, or managed service |
| **Codebase Capability Auditor Agent** | `agents/codebase-capability-auditor-agent.md` | You are about to add, update, or remove a plugin feature, or you need to verify what a plugin actually does before making any public claim |
| **SEO & Content Decision Agent** | `agents/seo-content-decision-agent.md` | You need to decide what page to create, update, merge, promote, delay, or avoid — requires the Growth Dashboard export |
| **Marketing & Documentation Agent** | `agents/marketing-documentation-agent.md` | You need to write a product page, service page, docs, FAQs, social posts, email copy, or any public-facing content — requires safe claims and a capability audit first |
| **Ops / Client Delivery Agent** | `agents/ops-client-delivery-agent.md` | You need to turn a service offer into a real delivery system — SOPs, onboarding, support boundaries, proposal language, or client communication templates |

**Flow order for new opportunities:**
Researcher → Product & Offer Alignment → Codebase Capability Auditor → SEO & Content Decision → Marketing & Documentation → Ops / Client Delivery

Not every task needs every agent. Start at the step that matches where you are.

---

## Step 1 — Start Every Session the Same Way

Paste this at the beginning of every Claude session, in any project:

```
Load the WebReadyNow knowledge base. Start with:
- 00-start-here.md
- agent-operating-rules.md
- knowledge-base-update-rules.md
- 01-core-context.md
- 02-current-priorities.md
- 03-safe-claims.md
- 04-offer-map.md

The task is: [one sentence describing what you want to do]
```

Claude will tell you which agent to load and what else is needed before starting.

---

## Step 2 — When You Are Not Sure Which Agent to Use

```
Load agent-router.md.
I want to: [describe your task in one sentence]
Which agent should I use, which prompt file should I load, 
and what inputs do I need before starting?
```

---

## Scenario A — Adding or Updating a Plugin Feature

### Before you write code

```
Load agents/codebase-capability-auditor-agent.md and follow it.

Plugin: [WRN Store Monitor / WRN Pricing Rules Pro / other]
I want to add or change: [describe the feature]

Audit the current codebase and tell me:
- What already exists related to this feature
- What is safe to build on
- What I need to build
- What claims will be safe once it is built
```

### After you write and test the code

```
Load agents/codebase-capability-auditor-agent.md and follow it.

Plugin: [plugin name]
New or updated feature: [feature name]
Codebase: [share the relevant files]

Audit only this feature. Classify each capability as:
implemented / partially implemented / missing / unsafe to claim / 
requires configuration / requires human service

Give me safe claims and unsafe claims for this feature.
```

**After the audit, update:**
- `06-products/[product].md` — add the new capability
- `03-safe-claims.md` — add approved claims, add blocked claims
- `02-current-priorities.md` — mark sprint or version complete if applicable

---

## Scenario B — Deciding Whether to Build Something New

### Step 1 — Research the idea

```
Load agents/researcher-agent.md and follow it.

Research objective: Should WebReadyNow build [idea]?

I need to know:
- Is there real buyer demand?
- What do competitors do?
- Does this support a plugin, a service, or both?
- What would the offer model be?
```

### Step 2 — Decide the offer model

```
Load agents/product-offer-alignment-agent.md and follow it.

Opportunity: [name]
Here is the researcher output: [paste the research findings]

Decide:
- What offer model fits? (plugin-only / service / productized service / 
  plugin plus setup / plugin plus managed service / custom)
- What needs to be built before this is viable?
- What are the safe claims?
```

**After alignment, update:**
- `04-offer-map.md` — add or update the offer
- `02-current-priorities.md` — add as a priority if decided
- Save alignment output to `09-agent-outputs/product-alignment/`

---

## Scenario C — SEO and Content Decisions

### Deciding what content to create, update, or fix

```
Load agents/seo-content-decision-agent.md and follow it.

Here is the Growth Dashboard export: [attach the Excel file]

I want to review: [service pages / blog articles / homepage / 
a specific cluster / all priority pages]

Run the source check and tell me what to create, update, 
merge, promote, delay, or avoid.
```

The agent will ask if anything is missing. Provide it before continuing.

**After the session, update:**
- Action Log CSV — paste the TSV rows the agent produced
- Content Inventory CSV — update changed pages
- Save decisions to `09-agent-outputs/seo-decisions/`

---

## Scenario D — Writing Copy, Docs, or Marketing Content

**Only use this after you have: safe claims confirmed + capability audit or SEO decision.**

```
Load agents/marketing-documentation-agent.md and follow it.

Sources available:
- Safe claims: [confirmed / attach 03-safe-claims.md section]
- Capability audit: [attach or paste audit output]
- SEO decision: [attach or paste if content is SEO-driven]

I need: [product page / service page / setup guide / FAQ / 
social post / email copy / feature explainer]

For: [product or service name]
Audience: [WooCommerce store owner / B2B buyer / etc.]
```

If required sources are missing, the agent will ask before writing. Do not push it to write without them.

**After the session:**
- Save drafts to `09-agent-outputs/marketing-docs/`
- After publishing: update Action Log CSV + Content Inventory CSV

---

## Scenario E — Designing a Service, SOP, or Delivery Process

```
Load agents/ops-client-delivery-agent.md and follow it.

Offer: [Managed Monitoring / Store Audit / WRN Store Monitor + Setup / etc.]
I need: [SOP / onboarding checklist / support boundaries / 
proposal language / client communication template]

Scope assumptions: [describe what is included and what is not]
Pricing / SLA: [state if known, or say unknown]
```

If scope or SLA is missing, the agent will flag it and produce an internal draft. Do not send internal drafts to clients.

**After the session:**
- Save to `09-agent-outputs/ops-delivery/`
- If service scope changed: update `07-services/service-map.md`
- If offer model changed: update `04-offer-map.md`

---

## Scenario F — Reviewing Growth Dashboard Data

```
Load agents/seo-content-decision-agent.md and follow it.

Here is the Growth Dashboard export: [attach]

This is a weekly review. Do not make final decisions yet.
Tell me:
- What patterns stand out in GSC and GA4?
- What pages are gaining or losing ground?
- What actions should I consider this week?
- What is the recommended priority?
```

---

## Step 3 — End Every Session with This

```
Apply the Ending Rule.

Tell me:
- What decision was made
- What files need to be updated
- What update level this is (Level 0 / 1 / 2 / 3)
- Whether a decision record is needed
- What output folder the result should be saved to
- What the next action is
```

---

## Step 4 — Commit After Every Meaningful Update

After updating any KB file, run:

```
git add [file names]
git commit -m "Describe what changed — example: Update Store Monitor safe claims after v1.12 audit"
git push
```

Good commit messages:
- `Add WRN Store Monitor alert retry feature — update safe claims and product file`
- `Update KB after care plans SEO optimization — Level 2`
- `Add SEO decision for WooCommerce integrations page`
- `Define WRN Store Monitor Plugin + Setup offer scope`

---

## Weekly Review Prompt

Run this at the start of each week or before any major session:

```
Load 02-current-priorities.md and 00-start-here.md.

Tell me:
- What TODOs are still open that may now be resolved?
- What KB files look stale based on recent work?
- What is the highest priority task right now?
- Is there anything I should update before starting new work?
```

---

## Quick Reference — Which Agent for What

| You want to… | Prompt file to load |
|---|---|
| Add or verify a plugin feature | `agents/codebase-capability-auditor-agent.md` |
| Decide whether to build something new | `agents/researcher-agent.md` → then `agents/product-offer-alignment-agent.md` |
| Decide what SEO/content to work on | `agents/seo-content-decision-agent.md` |
| Write a page, post, guide, or social post | `agents/marketing-documentation-agent.md` |
| Design a service, SOP, or delivery process | `agents/ops-client-delivery-agent.md` |
| Validate a new market or idea | `agents/researcher-agent.md` |
| Not sure which to use | Load `agent-router.md` first |

---

## Quick Reference — Which Files Update in Each Scenario

| Something changed | Update this |
|---|---|
| Safe claims changed | `03-safe-claims.md` |
| Product capability changed | `06-products/[product].md` |
| Offer model changed | `04-offer-map.md` |
| Service scope changed | `07-services/service-map.md` |
| Priority or sprint changed | `02-current-priorities.md` |
| Content created or updated | Action Log CSV + Content Inventory CSV |
| Any important decision made | Save to `09-agent-outputs/[folder]/` |
| Level 3 decision | Also create a decision record using `decision-record-template.md` |

---

## Rules That Protect the Framework

1. Do not write marketing copy without confirmed safe claims and a capability audit.
2. Do not make SEO decisions without the Growth Dashboard export.
3. Do not define a service offer without scope boundaries.
4. Do not skip straight to Marketing if Research or Alignment has not been done.
5. Do not update `03-safe-claims.md` without an audit to back it.
6. Do not let decisions live only in chat — save outputs and commit.
7. Use `TODO` for anything not yet confirmed. Never invent metrics, features, or proof.
