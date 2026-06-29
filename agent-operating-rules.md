# WebReadyNow — Agent Operating Rules

**Purpose:** Permanent operating rules for every Claude session, agent, or project working inside the WebReadyNow ecosystem. Load this file alongside the four core context files at the start of every session.

---

## 1. File Loading Rules

- Read `00-start-here.md` first, then `01-core-context.md`, `02-current-priorities.md`, `03-safe-claims.md`, and `04-offer-map.md`.
- Do not load the entire knowledge base unless explicitly asked.
- Load project-specific files (products, services, proof) only when the task requires them. See the Agent Reading Matrix in `00-start-here.md`.
- If a required file is missing or was not provided, ask for it before making a final decision. Do not fill gaps with assumptions.

---

## 2. Work Type Classification

Classify the current task before starting. This determines which files to load and what KB update level applies.

| Work Type | Description |
|---|---|
| SEO | Keyword decisions, content creation, title/meta changes, indexing, internal linking |
| Product | Plugin development, feature work, pricing, product documentation |
| Codebase Audit | Code review, performance analysis, plugin compatibility, technical debt |
| Marketing / Docs | Sales copy, landing pages, service descriptions, social content, documentation |
| Ops / Service Delivery | Client delivery, project scoping, SOPs, proposals |
| Research | Competitive research, market analysis, keyword research, discovery |
| Product Alignment | Connecting product features to buyer pain, offer positioning, pricing strategy |

---

## 3. Knowledge Base Update Level

After completing any task, classify whether the KB needs updating. Use the lowest level that accurately captures the change.

| Update Level | When to Use |
|---|---|
| **No KB update** | Tactical work with no lasting effect on KB accuracy (one-off answer, a draft, a small fix) |
| **Light KB update** | Changes that affect one page, one product record, or one content entry — update the relevant file only |
| **Full KB update** | Strategic changes that affect positioning, offers, safe claims, product capabilities, or multiple KB areas |

Do not update core KB files for every small action. A light change does not require touching `01-core-context.md`, `03-safe-claims.md`, or `04-offer-map.md`.

---

## 4. End-of-Task KB Update Check

At the end of every major task, state:

- What decision was made
- Which files should be updated, if any
- What output folder should store the result (see Agent Workflow, Section 9)
- Whether the KB update is light or full (no update / light / full)

Do not silently complete a task that changed product capabilities, safe claims, or offer positioning without flagging the relevant KB files.

---

## 5. Do Not Invent

Never invent:

- Metrics or performance figures not confirmed in the KB or provided by the user
- Product features or capabilities not documented in the relevant product file
- Case study outcomes or proof claims not in `08-proof/case-study-index.md`
- Delivery timelines or SLAs not backed by a defined SOP
- Safe claims not approved in `03-safe-claims.md`

Use `TODO` to mark anything that needs confirmation before use.

---

## 6. Respect Safe Claims

Check `03-safe-claims.md` before writing any product copy, service copy, or marketing content. Safe claims are non-negotiable — they exist because certain claims cannot be made without specific proof or conditions. Do not route around them.

---

## 7. Agency-First Rule

WebReadyNow is an agency first and a product builder second.

- Default to service-based framing unless the context is explicitly product-focused.
- Products (WRN Store Monitor, WRN Pricing Rules, WRN Hub) exist to support agency positioning and client outcomes — not the other way around.
- Do not make decisions that optimize for product growth at the expense of agency clarity, credibility, or service delivery.

---

## 8. Task-Specific Update Levels — SEO Example

Some work types have their own sub-classification systems that sit inside the broader KB update level framework. The SEO system is the reference example.

SEO work uses a three-level classification defined in `05-content-and-seo/seo-decision-rules.md`:

| SEO Level | What It Covers | Tracking Required |
|---|---|---|
| Level 1 — Tactical | Single change, one page, no new content (title, meta, alt text, internal link, CTA, indexing check) | Action Log only |
| Level 2 — Page Optimization | One page and its cluster; title, meta, content, and internal links may be updated; no new content or strategy change | Action Log + Content Inventory |
| Level 3 — Strategic | New page, new cluster, new offer, new content published, positioning change, safe claim change, or multiple pages restructured | Action Log + Content Inventory + relevant KB core/product/service files + decision record if needed |

Other work types should apply the same principle: use the lowest update level that accurately captures the change.

---

## 9. Agent Workflow

Six agents operate inside the WebReadyNow ecosystem. Each has a defined trigger, scope, and output folder. Run them in the order that the work requires — do not skip prerequisite agents.

---

### Agent 1 — Researcher Agent

**Use when:** Market evidence, buyer pain, competitor analysis, search opportunity, service or plugin validation, or external proof is needed.

**Inputs:** User prompt, external sources, Growth Dashboard exports if available.

**Outputs saved to:** `09-agent-outputs/researcher/`

---

### Agent 2 — Product & Offer Alignment Agent

**Use when:** Research is complete and a decision is needed on how to package or position an opportunity.

**Decides between:**
- Plugin only
- Service only
- Productized service
- Plugin plus setup
- Plugin plus managed service
- Custom implementation

**Outputs saved to:** `09-agent-outputs/product-alignment/`

---

### Agent 3 — Codebase Capability Auditor Agent

**Use before:** Making product claims, writing documentation, building product pages, writing sales pages, or creating SEO content that depends on actual plugin functionality.

**Verifies each claim or feature as one of:**
- Implemented
- Partial
- Missing
- Unsafe to claim
- Service-required

**Outputs saved to:** `09-agent-outputs/codebase-audits/`

---

### Agent 4 — SEO & Content Decision Agent

**Use after:** Source check is complete.

**Must ask for before proceeding:**
- Growth Dashboard export
- Content Inventory
- Action Log
- Safe Claims
- Offer Map
- Product or service context

**Outputs saved to:** `09-agent-outputs/seo-decisions/`

---

### Agent 5 — Marketing & Documentation Agent

**Use only after:** Safe claims are confirmed, feature inventory exists, and product alignment or SEO decision is in place.

**Writes:**
- Product pages
- Documentation
- FAQs
- Setup guides
- Sales copy
- Social posts
- Video scripts

**Outputs saved to:** `09-agent-outputs/marketing-docs/`

---

### Agent 6 — Ops / Client Delivery Agent

**Use when:** Turning offers into real service delivery.

**Creates:**
- SOPs
- Onboarding processes
- Support boundaries
- Incident processes
- Proposal language
- Client communication templates

**Outputs saved to:** `09-agent-outputs/ops-delivery/`

---

## 10. Sequencing Rule — No Skipping

Do not skip directly to marketing or SEO if research, offer alignment, or capability proof is missing.

**Required before Marketing & Documentation Agent (Agent 5):**
- Safe claims confirmed (`03-safe-claims.md`)
- Feature inventory confirmed (codebase audit output or product file)
- Product alignment decision exists, or an SEO decision exists

**Required before SEO & Content Decision Agent (Agent 4):**
- Growth Dashboard export provided
- Content Inventory available
- Offer Map and Safe Claims confirmed

If any required input is missing, ask for it or identify which prerequisite agent needs to run first.
