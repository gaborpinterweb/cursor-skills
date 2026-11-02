# secure-delivery

Cursor skill that audits the **current branch** for Acceptance and Regression
test coverage against the real diff. It proposes checks and where they should
live. It does **not** implement them unless you ask.

## When to use it

Ask Cursor to secure delivery, check evidence gaps before a PR, or run
`/secure-delivery`.

## How it works

1. Picks base branch: fresher of `main` / `master` (or a base you name), same
   rule as [`review-branch`](../review-branch/).
2. Reads the merge-base diff (including dirty tree unless you say otherwise).
3. Detects existing safety nets (`all-checks`, CI, linkchecker, html-validate,
   pa11y, i18n-guard, `node --test`, Playwright, etc.).
4. Lists up to 8 behavioral / user-visible outcomes from the diff.
5. Fills **Acceptance** (must for each outcome) and **Regression** (gaps only,
   or cite existing coverage).
6. Ends with `Ready for /create-pr` or `Blocked: missing Acceptance for …`.

## Definitions

| Kind | Meaning |
|---|---|
| Acceptance | Desired outcome is achieved |
| Regression | We did not break existing behavior |

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

`implement → /secure-delivery → /review-branch → /create-pr → /delivery-review`

Plan mode uses the User Rule in [`user-rules`](../user-rules/) to propose these
tests up front; this skill audits the branch after the work exists.
