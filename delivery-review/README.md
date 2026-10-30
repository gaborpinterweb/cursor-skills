# delivery-review

Cursor skill that produces a **delivery overview** for the open PR on the
current (or named) branch: code review, PR review, and Linear ticket review.
Three concise tables (`C#` / `P#` / `T#`) with blocking ⛔️ or warning ⚠️ rows.
Review-only; it does **not** apply fixes or edit the PR unless you ask.

Formerly `/review-pr` — that trigger still maps here.

## When to use it

Run `/delivery-review` (or `/review-pr`), or ask for a pre-merge delivery
overview before human review.

## How it works

1. Sync gate: local HEAD vs remote tip; asks you to sync if they differ.
2. Resolves the open PR for the branch with `gh`.
3. **Code** — branch/AGENTS compliance (same rules as [`review-branch`](../review-branch/), including silent CSS regressions); reuses a matching chat result when present; carries **Manual regression checks** when UI CSS/overlays moved.
4. **PR** — description, open feedback, and conversation vs the current diff.
5. **Ticket** — Linear IDs from PR/branch/commits vs implementation when Linear is usable.

## Output

Three tables, always, then Manual regression checks when the code pass requires them:

| Table | Numbers | Focus |
|-------|---------|-------|
| Code | C1… | Pattern / AGENTS / regressions (incl. silent CSS) |
| PR | P1… | Description & review feedback |
| Ticket | T1… | Linear AC vs diff |
| Manual checks | 1…8 | Browser / prod-vs-preview steps (when UI CSS/overlays moved) |

Columns: Number · Type (⛔️ / ⚠️) · Description · Reference · Suggestion.

**Code references are local** (`` `path:line` ``), never GitHub blobs. Diffable
rows append ` · vs origin/<base>`. Ask for details with `C2`, or `diff C1` for
the hunk against the PR base.

## Key files

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Standalone pre-merge check. For drafting a PR use [`create-pr`](../create-pr/).
For a code-only pass use [`review-branch`](../review-branch/). For bugs use
Bugbot; for vulns use Security Review.
