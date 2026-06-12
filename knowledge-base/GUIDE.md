# Knowledge Base — Guide

This folder is the agent's **persistent, per-product memory**. Without it, every QA session starts cold — the agent only knows the feature description and staging URL you hand it that session. With it, the agent understands a product's flows, rules, feature relationships, and historical weak spots *before* it writes a single test case.

## One knowledge base per product — keyed by Qase project code

If you test multiple products, their knowledge must stay separate — one product's business rules must never leak into another product's session. So each product gets its own folder, named by its **Qase project code** (the `QASE_PROJECT` set by the active profile):

```
knowledge-base/
├── _TEMPLATE/          ← copy this to start a new product KB
│   ├── product-flows.md
│   ├── business-rules.md
│   ├── feature-map.md
│   └── known-defects.md
├── SHOP/               ← Profile 1 (e.g. your e-commerce app) knowledge
├── CRM/                ← Profile 2 (e.g. your CRM) knowledge
└── BLOG/               ← Profile 3 (e.g. your docs portal) knowledge
```

At the start of any analysis session, the agent loads **only** `knowledge-base/<QASE_PROJECT>/` (see `CLAUDE.md` → Step 0.5). Skills in `.claude/agents/` stay global and shared — they are *how* to test. The knowledge base is *what* a specific product does.

## Start a knowledge base for a product

```bash
cp -r knowledge-base/_TEMPLATE knowledge-base/SHOP   # use your own Qase project code
```

Then open the four files and replace the example content with that product's real flows, rules, dependencies, and known defects. Even 3–4 real entries per file makes the agent noticeably sharper. If a product has no folder yet, the agent falls back to spec-only analysis — the KB is always additive, never required.

## The four files

| File | Answers | Used for |
|------|---------|----------|
| `product-flows.md` | "How does a user actually move through the product?" | Grounding happy-path and end-to-end test cases in real navigation |
| `business-rules.md` | "What is allowed, forbidden, or enforced?" | The bug-vs-intended oracle — a rule here is authoritative truth |
| `feature-map.md` | "What depends on what?" | Blast-radius / regression-impact awareness across features |
| `known-defects.md` | "Where has this product broken before?" | Probing weak spots; avoiding duplicate bug reports |

## How knowledge grows day by day

The knowledge base is meant to **compound** — every session should leave the agent smarter for the next one. Knowledge accumulates three ways:

1. **Automatically, after each session.** When WAY 1 finishes, `test-session-reporter` proposes knowledge-base updates for the active product — new confirmed defects → `known-defects.md`, new flows exercised → `product-flows.md`, new rules learned from the Jira ticket → `business-rules.md`. You approve the diff; it's written.
2. **On demand.** Tell the agent a fact ("the upload limit is now 20 MB") and it files it into the right product's KB.
3. **Bulk backfill.** Point the agent at a Jira epic or a set of closed bugs and it harvests rules and historical defects into the KB in one pass — so a mature product gets a rich KB quickly instead of building from zero.

After a month of testing one product, its folder becomes a genuine product encyclopedia — and it never bleeds into another product.

## Privacy note

These files contain a product's business logic. The `_TEMPLATE` content is generic and safe to commit. If your real product KBs are sensitive, gitignore the product folders (keep `_TEMPLATE/` and this guide for others):

```gitignore
knowledge-base/*/
!knowledge-base/_TEMPLATE/
```
