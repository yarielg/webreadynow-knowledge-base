# Agent Router

## Purpose

This file explains which WebReadyNow agent to use depending on the task, what inputs each agent requires, and where outputs should be saved.

---

## Agent Flow

Default strategic flow:

```
Researcher Agent
→ Product & Offer Alignment Agent
→ Codebase Capability Auditor Agent
→ SEO & Content Decision Agent
→ Marketing & Documentation Agent
→ Ops / Client Delivery Agent
```

Not every task needs every agent. Use the Quick Routing Rules below to find the right starting point.

When using an agent, load the matching prompt file from `agents/` instead of rewriting instructions manually.

---

## Agent 1 — Researcher Agent

Use when exploring:

- New market opportunity
- Buyer pain
- Competitor landscape
- SEO opportunity validation
- Plugin idea validation
- Service idea validation
- External proof
- Pricing and positioning research
- Industry patterns

**Reusable prompt:** `agents/researcher-agent.md`

**Output folder:** `09-agent-outputs/researcher/`

---

## Agent 2 — Product & Offer Alignment Agent

Use after research or when deciding how to package an opportunity.

Decides between:

- Plugin-only
- Service-only
- Productized service
- Plugin plus setup
- Plugin plus managed service
- Custom implementation

**Reusable prompt:** `agents/product-offer-alignment-agent.md`

**Output folder:** `09-agent-outputs/product-alignment/`

---

## Agent 3 — Codebase Capability Auditor Agent

Use before making product claims, writing product pages, documentation, sales pages, or SEO content that depends on actual plugin functionality.

Verifies each claim or feature as:

- Implemented
- Partially implemented
- Missing
- Unsafe to claim
- Service-required
- Roadmap needed

**Reusable prompt:** `agents/codebase-capability-auditor-agent.md`

**Output folder:** `09-agent-outputs/codebase-audits/`

---

## Agent 4 — SEO & Content Decision Agent

Use when deciding:

- What page to create
- What page to update
- What article to write
- What page to delay
- What content to avoid
- What internal links to add
- What CTA to use

**Required sources before proceeding:**

- Latest Growth Dashboard Excel export
- Content Inventory
- Action Log
- Safe Claims (`03-safe-claims.md`)
- Offer Map (`04-offer-map.md`)
- Relevant product or service context

**Reusable prompt:** `agents/seo-content-decision-agent.md`

**Output folder:** `09-agent-outputs/seo-decisions/`

---

## Agent 5 — Marketing & Documentation Agent

Use only after safe claims are confirmed, feature inventory exists, and a product alignment or SEO decision is in place.

Creates:

- Product pages
- Service pages
- Documentation
- FAQs
- Setup guides
- Sales copy
- Social posts
- Email copy
- Video scripts

**Reusable prompt:** `agents/marketing-documentation-agent.md`

**Output folder:** `09-agent-outputs/marketing-docs/`

---

## Agent 6 — Ops / Client Delivery Agent

Use when turning offers into real service delivery.

Creates:

- SOPs
- Onboarding steps
- Support boundaries
- Incident workflows
- Proposal language
- Monthly report templates
- Client communication templates

**Reusable prompt:** `agents/ops-client-delivery-agent.md`

**Output folder:** `09-agent-outputs/ops-delivery/`

---

## Quick Routing Rules

| Situation | Start With |
|---|---|
| New opportunity or market question | Researcher Agent |
| Plugin vs. service vs. productized decision | Product & Offer Alignment Agent |
| Product claim, product page, or feature documentation | Codebase Capability Auditor Agent |
| SEO or page decision | SEO & Content Decision Agent |
| Sales copy, docs, FAQs, social posts | Marketing & Documentation Agent |
| Client delivery, SOPs, proposals, onboarding | Ops / Client Delivery Agent |
| Small existing SEO edit (title, meta, internal link) | SEO Agent or direct execution — no Researcher needed |

---

## Do Not Skip Rules

- Do not jump to marketing if safe claims are missing.
- Do not publish SEO content based on product claims without a capability audit.
- Do not define a service offer without service boundaries or delivery assumptions.
- Do not create content just because a keyword has impressions.
- Do not skip directly to marketing or SEO if research, offer alignment, or capability proof is missing.
