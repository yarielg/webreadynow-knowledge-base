# How to Use the WebReadyNow AI Team

This file explains how to use the WebReadyNow AI team for agency growth, product strategy, plugin development, SEO, marketing, documentation, and service delivery.

The goal is to keep every Claude/AI session aligned with the same WebReadyNow strategy without re-explaining the business every time.

---

## Core Rule

WebReadyNow is a WooCommerce-focused agency first and a product builder second.

Plugins, content, landing pages, documentation, audits, and service workflows should support WebReadyNow’s larger business goals:

- Better WooCommerce outcomes for store owners
- Clear product/service positioning
- Safe and accurate claims
- More qualified leads
- More repeatable service delivery
- Stronger plugin-supported agency offers

Do not create isolated ideas that do not connect back to WebReadyNow’s agency growth strategy.

---

## Always-Load Files

For strategic, product, SEO, marketing, documentation, or service decisions, start with these files:

1. `agent-operating-rules.md`
2. `knowledge-base-update-rules.md`
3. `01-core-context.md`
4. `02-current-priorities.md`
5. `03-safe-claims.md`
6. `04-offer-map.md`

Load additional files only when needed.

---

## Conditional Files

Use these only when relevant:

- `agent-router.md` — when choosing the correct agent or starting a new task type.
- `decision-record-template.md` — when recording an important decision.
- `05-content-and-seo/seo-decision-rules.md` — for SEO/content decisions.
- `06-products/*.md` — when working on a specific WebReadyNow product.
- `07-services/*.md` — when working on a service or productized service.
- `09-agent-outputs/` — when reviewing previous agent work.
- `10-projects/` — when a specific project has its own history or context.

Do not load the entire knowledge base unless there is a clear reason.

---

## Agent Router

Use the agent router when the correct agent is unclear:

`agent-router.md`

Each agent has its own prompt file in:

`agents/`

Do not rewrite agent instructions manually. Load the correct prompt file.

---

## AI Team Roles

### 1. Researcher Agent

Prompt file:

`agents/researcher-agent.md`

Use for:

- Market research
- Competitor analysis
- Buyer pain research
- WooCommerce opportunity research
- SEO opportunity validation
- Plugin idea validation
- Service opportunity validation
- Forum/review mining
- Commercial intent research

Use this agent when the question is:

> Is this opportunity real, valuable, and worth pursuing?

Expected output:

- Buyer pain signals
- Market evidence
- Competitor patterns
- Commercial intent
- SEO/content opportunity
- Plugin opportunity
- Service opportunity
- Recommendation
- Next agent

---

### 2. Product & Offer Alignment Agent

Prompt file:

`agents/product-offer-alignment-agent.md`

Use for:

- Deciding whether something should be a plugin, service, productized service, setup package, managed service, or custom implementation.
- Clarifying product vs service boundaries.
- Packaging a feature or capability into an offer.
- Deciding how a plugin supports WebReadyNow services.

Use this agent when the question is:

> How should WebReadyNow package this opportunity?

Expected output:

- Recommended offer model
- Plugin role
- Service role
- Setup/managed service opportunity
- Safe claims
- Claims to avoid
- Pricing/packaging notes
- Next agent

---

### 3. Codebase Capability Auditor Agent

Prompt file:

`agents/codebase-capability-auditor-agent.md`

Use for:

- Verifying what a plugin/theme/codebase actually does.
- Checking if a product claim is safe.
- Separating implemented features from partial, UI-only, missing, or unsafe features.
- Auditing launch readiness.
- Auditing plugin integrations.

Use this agent when the question is:

> What can we safely claim based on the actual code?

Expected output:

- Implemented capabilities
- Partially implemented capabilities
- UI-only/stub capabilities
- Missing capabilities
- Requires configuration
- Requires human service
- Safe claims
- Unsafe claims
- Launch blockers
- Development roadmap recommendations

Do not publish product, SEO, or marketing claims before this agent verifies the capability.

---

### 4. SEO & Content Decision Agent

Prompt file:

`agents/seo-content-decision-agent.md`

Use for:

- Deciding what page or article to create/update.
- Reviewing Growth Dashboard exports.
- Interpreting GA4/GSC signals.
- Choosing SEO actions.
- Updating Action Log / Content Inventory.
- Deciding whether a content action is Level 1, 2, or 3.

Use this agent when the question is:

> What SEO/content action should WebReadyNow take next?

Expected output:

- SEO decision
- Target page/content
- Reasoning
- Safe claims check
- Action Log row
- Content Inventory update if needed
- KB update level
- Next action

---

### 5. Marketing & Documentation Agent

Prompt file:

`agents/marketing-documentation-agent.md`

Use for:

- Product pages
- Service pages
- Product documentation
- Setup guides
- FAQs
- Sales copy
- Launch copy
- Social posts
- Video scripts
- In-plugin help text

Use this agent only after safe claims are clear.

Use this agent when the question is:

> How should we explain this clearly and safely to customers?

Expected output:

- Draft content
- Safe claims used
- Claims avoided
- CTA recommendations
- Internal link recommendations
- Documentation structure
- Implementation notes

Do not use this agent to invent capabilities. It writes from verified claims.

---

### 6. Ops / Client Delivery Agent

Prompt file:

`agents/ops-client-delivery-agent.md`

Use for:

- SOPs
- Client onboarding
- Support boundaries
- Productized service workflows
- Managed service workflows
- Incident response workflows
- Monthly report templates
- Scope definitions
- Handoff checklists
- Client communication templates

Use this agent when the question is:

> How do we deliver this consistently for clients?

Expected output:

- Delivery workflow
- Included scope
- Excluded scope
- Required inputs
- Internal process
- Client communication
- Escalation rules
- Support boundaries
- Draft SOP/checklist/template

---

## Standard Workflow

Use this workflow for most important WebReadyNow decisions:

1. **Research**
    - Use Researcher Agent if the opportunity needs validation.

2. **Package**
    - Use Product & Offer Alignment Agent to decide plugin/service/setup/managed/custom model.

3. **Verify**
    - Use Codebase Capability Auditor Agent before making product or feature claims.

4. **Position**
    - Use Marketing & Documentation Agent after claims are verified.

5. **Publish / Track**
    - Use SEO & Content Decision Agent when the action affects pages, content, SEO, GA4/GSC, Action Log, or Content Inventory.

6. **Deliver**
    - Use Ops / Client Delivery Agent when the work becomes a client workflow, SOP, productized service, or managed service.

---

## Common Routing Examples

### New plugin idea

Use:

1. Researcher Agent
2. Product & Offer Alignment Agent
3. Codebase Capability Auditor Agent when code exists
4. Marketing & Documentation Agent only after claims are safe

---

### Existing plugin feature claim

Use:

1. Codebase Capability Auditor Agent
2. Update `03-safe-claims.md`
3. Marketing & Documentation Agent if copy/docs are needed

---

### New service page

Use:

1. Product & Offer Alignment Agent
2. SEO & Content Decision Agent
3. Marketing & Documentation Agent
4. Ops / Client Delivery Agent if service delivery needs structure

---

### SEO opportunity from Growth Dashboard

Use:

1. SEO & Content Decision Agent
2. Marketing & Documentation Agent if copy needs to be drafted
3. Update Action Log / Content Inventory based on the SEO update level

---

### Client delivery workflow

Use:

1. Ops / Client Delivery Agent
2. Product & Offer Alignment Agent if the workflow may become a productized service
3. Marketing & Documentation Agent if client-facing copy is needed

---

## Knowledge Base Update Levels

Use the lowest update level needed.

### Level 0 — No KB Update

Use when:

- The task was exploratory.
- No decision was made.
- No source-of-truth file changed.
- No future agent needs the information.

### Level 1 — Execution Tracker Only

Use when:

- The action affects tracking only.
- Update Action Log or task tracker, but not core KB.

### Level 2 — Light KB Update

Use when:

- A product, service, safe claim, audit, or content status changed.
- Update the relevant product/service/safe-claims file.
- Save the agent output if useful.

### Level 3 — Full KB Update / Decision Record

Use when:

- A major positioning, offer, product, service, or architecture decision was made.
- Update relevant KB files.
- Create a decision record using `decision-record-template.md`.

---

## Ending Rule for Every Important AI Session

Every important session should end with:

- Decision made
- Agent used
- Source files reviewed
- Files/pages/products affected
- Safe claims used or changed
- Claims avoided
- KB update level
- Files to update
- Whether a decision record is needed
- Recommended next action

No important decision should live only inside chat.

---

## What Not To Do

Do not:

- Create more agents unless there is a repeated, clearly separate workflow.
- Publish marketing claims before code capability audit.
- Position internal infrastructure as a sellable product unless explicitly decided.
- Duplicate full agent prompts inside router or core files.
- Update unrelated files during a focused task.
- Load the entire KB for small tasks.
- Create content only because a keyword has impressions.
- Invent metrics, guarantees, features, proof, or customer outcomes.
- Blur plugin-only value with managed service value.

---

## Current AI Team Status

The active WebReadyNow AI team is:

1. Researcher Agent
2. Product & Offer Alignment Agent
3. Codebase Capability Auditor Agent
4. SEO & Content Decision Agent
5. Marketing & Documentation Agent
6. Ops / Client Delivery Agent

This is the complete core team for now.

Do not add more agents unless the workflow becomes repeatedly necessary and cannot be handled by one of these six.

---

## Simple Rule

When unsure, ask:

1. Is this research?
2. Is this offer packaging?
3. Is this a claim that needs code verification?
4. Is this SEO/content?
5. Is this customer-facing copy/docs?
6. Is this client delivery/operations?

Then use the matching agent.