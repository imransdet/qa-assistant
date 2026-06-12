# Knowledge Base

This folder is the agent's **persistent product memory**. Without it, every QA session starts cold — the agent only knows the feature description and staging URL you hand it that session. With it, the agent understands your product's flows, rules, feature relationships, and historical weak spots *before* it writes a single test case.

The agent loads these files automatically (see `CLAUDE.md` → Step 0.5) at the start of any session that analyzes requirements (WAY 1, WAY 3, WAY 4).

## The four files

| File | Answers | Used for |
|------|---------|----------|
| `product-flows.md` | "How does a user actually move through the product?" | Grounding happy-path and end-to-end test cases in real navigation |
| `business-rules.md` | "What is allowed, forbidden, or enforced?" | The bug-vs-intended oracle — a rule here is authoritative truth |
| `feature-map.md` | "What depends on what?" | Blast-radius / regression-impact awareness across features |
| `known-defects.md` | "Where has this product broken before?" | Probing weak spots; avoiding duplicate bug reports |

## How to maintain it

- **Start small.** Even 3–4 entries per file makes the agent noticeably sharper. You do not need to document the whole product on day one.
- **Update after each session.** When the agent finds a real bug, add it to `known-defects.md`. When a flow changes, update `product-flows.md`.
- **Keep rules atomic.** One rule per `BR-xx` row. The agent cites these IDs when it flags a defect.
- **This is product knowledge, not secrets.** No API tokens or credentials here. If your business rules are sensitive, gitignore this folder (see note below).

## Privacy note

These files contain your product's business logic. The example content shipped here is generic and safe to commit. If you replace it with real proprietary rules and don't want them public, add `knowledge-base/` to `.gitignore` (keeping a `knowledge-base/README.md` and the templates is still useful for others).
