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
3. **Code** — branch/AGENTS compliance (same rules as [`review-branch`](../review-branch/)); reuses a matching chat result when present.
4. **PR** — description, open feedback, and conversation vs the current diff.
5. **Ticket** — Linear IDs from PR/branch/commits vs implementation when Linear is usable.

## Output

Three tables, always:

| Table | Numbers | Focus |
|-------|---------|-------|
| Code | C1… | Pattern / AGENTS / regressions |
| PR | P1… | Description & review feedback |
| Ticket | T1… | Linear AC vs diff |

Columns: Number · Type (⛔️ / ⚠️) · Description · Reference · Suggestion.

Ask for details by number (e.g. `expand C2`).

## Key files

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Standalone pre-merge check. For drafting a PR use [`create-pr`](../create-pr/).
For a code-only pass use [`review-branch`](../review-branch/). For bugs use
Bugbot; for vulns use Security Review.
