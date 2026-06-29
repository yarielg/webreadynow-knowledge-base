# Agent Operating Rules

## Purpose

This file defines how Claude sessions and WebReadyNow agents must operate across all projects.

---

## Core Loading Rule

Agents must not load the entire knowledge base by default.

For most sessions, read these files first:

- `00-start-here.md`
- `01-core-context.md`
- `02-current-priorities.md`
- `03-safe-claims.md`
- `04-offer-map.md`
- `agent-router.md`
- `knowledge-base-update-rules.md`

Then read only the specific files needed for the task. See the Agent Reading Matrix in `00-start-here.md`.

---

## WebReadyNow Business Rule

WebReadyNow is a WooCommerce agency first and product builder second.

Every decision should support:

- Agency profitability
- WooCommerce service revenue
- Plugin and product revenue
- Productized services
- Stronger positioning
- Safer claims
- Better client delivery
- Coherent WebReadyNow brand direction

Products (WRN Store Monitor, WRN Pricing Rules, WRN Hub) exist to support agency positioning and client outcomes. Do not optimize for product growth at the expense of agency clarity, credibility, or service delivery.

---

## Source-of-Truth Rule

Agents must make decisions from confirmed sources:

- Knowledge base core files
- Product and service files
- Safe claims (`03-safe-claims.md`)
- Capability audits
- Growth Dashboard exports
- Content Inventory
- Action Log
- Researcher outputs
- Product Alignment outputs
- Codebase Auditor outputs

Agents must not pretend to have reviewed files that were not provided or loaded.

---

## Missing Context Rule

If required files or inputs are missing, ask for them before making a final decision. Do not fill gaps with assumptions.

Examples:

- SEO decision requires: Growth Dashboard export, Content Inventory, Action Log, Safe Claims, Offer Map.
- Product page or marketing claim requires: Safe Claims and Capability Audit.
- New opportunity requires: Researcher output.
- Packaging decision requires: Product & Offer Alignment output.
- Service delivery process requires: Service Map and SOP or support boundary context.

---

## Safe Claim Rule

Never recommend content, marketing, documentation, or service promises that depend on unsupported product or service claims.

Use safer language:

- Helps detect
- Surfaces
- Alerts when thresholds are crossed
- Logs events
- Supports diagnosis
- Provides diagnostic context
- Helps identify patterns

Avoid unless verified:

- Automatically fixes
- Guarantees
- Prevents all revenue loss
- Detects every issue
- Monitors the full journey
- Replaces human support

Check `03-safe-claims.md` before writing any product copy, service copy, or marketing content.

---

## Task Classification Rule

Every session must identify the task type before starting:

- Research
- Product & Offer Alignment
- Codebase Capability Audit
- SEO & Content Decision
- Marketing & Documentation
- Ops / Client Delivery
- General strategy
- Execution task

This determines which agent to use and which files to load. See `agent-router.md`.

---

## Ending Rule

At the end of every important task, agents must report:

- Decision made
- Agent used or recommended
- Sources reviewed
- Files or pages affected
- Recommended knowledge base update level
- Files that should be updated
- Whether a decision record is needed
- Next action

---

## No-Chaos Rule

- Do not update unrelated files.
- Do not create unnecessary new files.
- Do not split small topics into too many files.
- Do not read the whole knowledge base unless explicitly asked.
- Do not invent metrics, features, case study outcomes, revenue figures, product claims, or service promises.
- Use `TODO` to mark anything that needs confirmation before use.
