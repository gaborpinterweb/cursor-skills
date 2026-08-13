# create-pr

Cursor skill that **drafts** a pull request title and body, asks for confirmation, then creates the PR with `gh`. It does **not** open a PR until you approve the message (and any links you add).

## When to use it

Ask Cursor to create a PR, open a pull request, or run `/create-pr`.

## How it works

1. Gathers branch status, diff vs base, and commit history (same fresher `main`/`master` base rule as [`review-branch`](../review-branch/) unless you name a base).
2. Extracts ticket IDs from commits/branch (Jira, Linear, GitHub Issues, etc.) and resolves ticket/preview URLs when possible.
3. Drafts title + body with:
   - **Changes** — max 8 bullets
   - **Decisions** — max 8 bullets
   - **Links** — ticket + preview (or prompts you to supply them)
4. Shows the draft and waits for confirmation or edits.
5. On approval: pushes if needed, runs `gh pr create`, returns the PR URL.

## Confirmation gate

Missing preview or ticket URL → skill suggests including them and waits. No invented IDs or URLs.

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Often run after [`review-branch`](../review-branch/) once Critical errors are cleared.
