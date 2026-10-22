# explain-changes

Cursor skill that **explains** the intention and behaviour of diffs for a
user-supplied file list against `origin/main` or `origin/master`. It does
**not** review for bugs or compliance.

## When to use it

Ask Cursor to explain changes in specific files, or run `/explain-changes`
with a file list.

## How it works

1. Resolves base as the fresher of `origin/main` / `origin/master` (same rule
   as [`review-branch`](../review-branch/)), unless you name a base.
2. Diffs **only** the files you listed.
3. Summarises intention, then a numbered list of behavioural units with
   editor-navigable `path:line` links.

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Useful before [`review-branch`](../review-branch/) when you want to understand
what the branch does in a few files, without a compliance pass.
