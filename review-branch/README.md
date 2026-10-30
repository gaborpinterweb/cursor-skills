# review-branch

Cursor skill for a **pattern / AGENTS compliance** review of the current branch. It grades the diff against repo conventions — not a Bugbot or Security Review substitute. Review-only; it does **not** apply fixes unless you ask.

## When to use it

Ask Cursor to review the branch, run `/review-branch`, or do a pre-merge compliance check before human review.

## How it works

1. Picks base branch: fresher of `main` / `master` (or a base you name).
2. Reads `README.md`, `AGENTS.md`, and local patterns for **changed paths only** (and competing CSS consumers when selectors were re-scoped).
3. Reviews the merge-base diff (including dirty tree unless you say otherwise).
4. Replies with `Base: …` plus two capped tables (8 rows each), with IDs and a **Recommended fix** column for follow-ups:
   - **Critical errors** — `1.1`, `1.2`, … (must-fix, evidenced)
   - **Watchouts** — `2.1`, `2.2`, … (eyebrow-raisers for leads/devs)
5. When UI CSS/markup or interactive overlays moved: **Manual regression checks** (up to 8 steps you can run in the browser / vs prod).

## What it checks

Core guidelines: no regressions, no accidental globals, reuse partials/components, naming, no magic values/colors, repo patterns, no dead code, respect README/AGENTS.

Hugo/Astro-oriented definitions for **regression** and **global change**, including **silent CSS regressions** (cascade, specificity after scope lifts, layout overflow clipping, theme/surface AA). Diff-scoped extras (a11y, i18n, CI, secrets/GDPR) only when the diff touches those surfaces.

Every finding needs a **Reference** (AGENTS section, existing file/pattern, or unchanged competing consumer).

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Use after [`review-understand`](../review-understand/) when you first need to parent AI output, then before [`create-pr`](../create-pr/). For bugs use Bugbot; for vulns use Security Review. For a delivery overview (PR + ticket) use [`delivery-review`](../delivery-review/).
