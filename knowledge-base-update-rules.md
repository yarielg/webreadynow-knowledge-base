# Knowledge Base Update Rules

## Purpose

This file explains when and how WebReadyNow knowledge base files should be updated after an agent task.

---

## Core Rule

No important decision should live only inside chat.

Important decisions should be recorded in the right place. But not every small task requires a full knowledge base update. Use the lowest update level that accurately captures the work.

---

## Update Levels

### Level 0 — No Knowledge Base Update

Use for:

- Casual discussion
- Temporary idea
- Small clarification
- No decision made
- No action taken

**Tracking required:** None.

---

### Level 1 — Execution Tracker Only

Use for:

- Tiny tactical task
- One small page edit
- One internal link
- One meta description tweak
- One typo fix
- One indexing check
- One small implementation step

**Tracking required:** Action Log or task tracker only.

---

### Level 2 — Light Knowledge Base Update

Use for:

- Existing page optimization
- Existing product or service page adjustment
- One page and its related cluster affected
- Content Inventory changed
- SEO title, meta, content, or internal links changed
- No new content created
- No new offer or strategy change

**Tracking required:** Action Log + Content Inventory + relevant summary file if useful.

**Examples:**
- Updating a service page SEO title and meta to match a new search intent.
- Optimizing an existing service page for CTR.
- Adding important internal links across a content cluster.

---

### Level 3 — Full Knowledge Base Update

Use for:

- New page created
- New article published
- New content cluster created
- New offer
- Product or service positioning changed
- Safe claim changed
- Product roadmap changed
- Capability audit changed public claims
- Service SOP or delivery model changed
- Multiple pages restructured

**Tracking required:** Action Log + Content Inventory + relevant core, product, service, or SEO files + decision record + agent output saved if applicable.

---

## SEO-Specific Level Rules

For SEO-specific Level 1, 2, and 3 definitions, see `05-content-and-seo/seo-decision-rules.md`.

---

## Agent Ending Requirement

For required end-of-task reporting, see `agent-operating-rules.md` (Ending Rule section).

---

## Do Not Over-Update Rule

Do not update core files for every small task.

- A meta description tweak does not update `04-offer-map.md`.
- An internal link does not update `03-safe-claims.md`.
- A title optimization does not update product roadmap files.
- A new product claim does update `03-safe-claims.md` and the relevant product file.

---

## Commit Rule

After meaningful knowledge base updates:

```bash
git status
git add .
git commit -m "Describe KB update"
git push
```

Use clear commit messages:

- `Add agent governance rules`
- `Update KB after care plans SEO optimization`
- `Update Store Monitor safe claims after audit`
- `Add SEO decision for integrations page`
