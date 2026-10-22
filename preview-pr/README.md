# preview-pr

Cursor skill that **previews an open PR** for the current (or named) branch:
sync gate, resolve via `gh`, optional reuse of `/review-branch` from chat, then
align PR description and open feedback against the diff. Optionally reviews
Linear tickets vs implementation after you confirm a fetch. Review-only; it does
**not** apply fixes or edit the PR unless you ask.

## When to use it

Ask Cursor to preview a PR, run `/preview-pr`, or do a pre-merge PR context
check before human review.

## How it works

1. Compares local HEAD to the remote tip; asks you to sync if they differ.
2. Finds open PRs for the branch with `gh`. Stops if none; asks which if several.
3. Looks in **this chat** for a `/review-branch` result. Reuses it if present;
   otherwise asks whether to run `/review-branch` first (never auto-runs it).
4. Reviews PR **alignment**: description vs diff, open review feedback, and PR
   conversation constraints.
5. If Linear is connected and ticket IDs appear, lists them and **asks** before
   fetching; on yes, reviews ticket AC vs the implementation.

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Often after [`review-branch`](../review-branch/) in the same chat (so that
section can be reused), and before merge. For drafting a PR use
[`create-pr`](../create-pr/). For bugs use Bugbot; for vulns use Security Review.
