# WebReadyNow Knowledge Base — Start Here

This knowledge base is the central source of truth for WebReadyNow. Agents should read the core files first, then only the specific files needed for the task. Do not load the entire knowledge base unless explicitly asked.

---

## What This Is

A centralized Markdown knowledge base shared across all WebReadyNow Claude sessions and projects. Every plugin, service page, SEO decision, product idea, and client delivery connects back to the same business goal:

**Make WebReadyNow profitable, coherent, professional, and focused on WooCommerce outcomes.**

---

## Core Files — Read These First (Every Session)

| File | What It Contains |
|---|---|
| `agent-operating-rules.md` | How all agents must operate — load every time |
| `knowledge-base-update-rules.md` | When and how to update the knowledge base — load every time |
| `01-core-context.md` | Agency identity, business model, what WebReadyNow sells |
| `02-current-priorities.md` | What is being worked on right now |
| `03-safe-claims.md` | What we can and cannot claim — global rules |
| `04-offer-map.md` | All products, services, and offer models |

Read these six files by default. Then read only the files the task requires.

---

## Conditional Files — Load Only When Needed

| File | Load When |
|---|---|
| `agent-router.md` | Starting a new task type or unsure which agent to use |
| `decision-record-template.md` | Creating a decision record for an important decision |
| `06-products/wrn-store-monitor.md` | Working on WRN Store Monitor plugin, marketing, or monitoring service |
| `06-products/wrn-pricing-rules.md` | Working on WRN Pricing Rules / Price List Manager |
| `06-products/wrn-hub.md` | Working on WRN Hub |
| `06-products/webreadynow-theme.md` | Working on WebReadyNow theme |
| `07-services/service-map.md` | Writing service pages, proposals, or CTAs |
| `08-proof/case-study-index.md` | Writing marketing copy, case studies, or proof claims |
| `05-content-and-seo/seo-decision-rules.md` | Making any SEO or content decision |
| `05-content-and-seo/content-inventory-summary.md` | Checking what content already exists |

---

## Agent Reading Matrix

Six agents operate inside this ecosystem. Each maps to a work type and a set of required files. See `agent-router.md` for full agent definitions, triggers, output folders, and quick routing rules.

| Agent / Work Type | Core Files | Additional Files |
|---|---|---|
| Researcher Agent | 01, 02, 03, 04 | Relevant product or service file based on research scope |
| Product & Offer Alignment Agent | 01, 03, 04 | relevant product file, offer-map.md, service-map.md |
| Codebase Capability Auditor Agent | 01, 03, 04 | Relevant product file |
| SEO & Content Decision Agent | 01, 02, 03, 04 | seo-decision-rules.md, content-inventory-summary.md, case-study-index.md, relevant product file |
| Marketing & Documentation Agent | 01, 03, 04 | Relevant product file, case-study-index.md, service-map.md |
| Ops / Client Delivery Agent | 01, 04 | service-map.md |

---

## Key Rules

WebReadyNow is an agency first and product builder second. Do not invent metrics, claims, or capabilities. Check `03-safe-claims.md` before any copy is written.

For full agent behavior rules, see `agent-operating-rules.md`.

---

## Agent Governance

Core governance files are loaded every session: `agent-operating-rules.md` and `knowledge-base-update-rules.md`.

Load `agent-router.md` when starting a new task type or when the correct agent is unclear. Load `decision-record-template.md` only when creating a decision record.

Agents must not load the whole knowledge base by default.
