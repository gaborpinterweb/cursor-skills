---
name: secure-delivery
description: >-
  Audits the current branch vs main/master for Acceptance and Regression test
  coverage gaps; proposes checks and where they should live. Use when the user
  runs /secure-delivery, asks to secure delivery, or wants evidence gaps before
  opening a PR.
disable-model-invocation: true
---

# Secure Delivery

Branch-level **Acceptance / Regression** coverage audit against the real diff.
Propose checks and placement. Do **not** implement checks unless the user
explicitly asks.

Not a substitute for `review-branch` (AGENTS/pattern compliance),
`delivery-review` (open PR C/P/T overview), or Bugbot / Security Review.

## Definitions

- **Acceptance**: proves the desired outcome is achieved. Required for each
  behavioral or user-visible change in the diff.
- **Regression**: proves we did not break existing behavior. Propose only when
  existing coverage does not already cover the new risk; otherwise cite what
  covers it.

## Base branch

Default base is `main` or `master`, whichever is **fresher**.

1. Prefer remote-tracking refs (`origin/main`, `origin/master`); else local.
2. If only one exists, use that.
3. If both exist, pick the tip with the newer committer date:
   `git log -1 --format=%cI <ref>`
4. If neither exists, use `git symbolic-ref refs/remotes/origin/HEAD` or ask.
5. User-named base wins (`against develop`, `vs release/x`).

Diff scope: merge-base of HEAD and base through working tree (committed + staged
+ unstaged), unless the user asks for commits-only or uncommitted-only.

```bash
git merge-base HEAD <base>
git diff --stat <merge-base>...HEAD
git diff <merge-base>
git status --porcelain
```

**User-facing base note:** one short line only, e.g. `Base: origin/main`.
Do not explain the fresher-ref logic in the reply.

## Workflow

1. Resolve base; print `Base: …`.
2. Collect the merge-base diff (include dirty tree unless told otherwise).
3. Detect existing safety nets in the repo (touched site package + root):
   `all-checks`, `package.json` scripts, CI workflows, `html-validate`,
   linkchecker, pa11y, i18n-guard, `node --test`, Playwright, wording-check,
   and similar. Prefer naming real script/job names from the tree.
4. Extract up to **8** behavioral / user-visible outcomes from the diff (not a
   file list). Pure docs/content with no behavioral change: one-line skip and
   a single None row where a table is required; do not invent outcomes.
5. For each outcome, fill **Acceptance** (status `present` / `missing`, check,
   where it should live).
6. Fill **Regression**: `covered` citing existing gates, or `gap` with a
   proposed check. Cap **8** gap-oriented rows. Do not invent parallel suites
   when existing `all-checks` / CI already covers the risk.
7. Placement defaults:
   - Prefer site `all-checks` / CI for build and HTML crawl checks
   - `node --test` (or equivalent) for pure helpers / fixtures
   - Thin Playwright only for layout, `[hidden]`, overflow, embeds, JS paths
   - Avoid git hooks for build-heavy checks
8. End with readiness: `Ready for /create-pr` or
   `Blocked: missing Acceptance for …`.
9. Close briefly: review file diffs manually; you own commits and messages.

## Output format

```text
Base: origin/main

### Outcomes
| # | Outcome | From diff |
|---|---------|-----------|
| 1 | … | `path` |

### Acceptance
| # | Outcome | Status | Check | Where |
|---|---------|--------|-------|-------|
| A1 | … | missing / present | … | all-checks / node --test / … |

### Regression
| # | Risk | Status | Coverage or proposed check | Where |
|---|------|--------|----------------------------|-------|
| R1 | … | covered / gap | existing script or proposal | … |

Ready for /create-pr | Blocked: …
```

**Empty / skip:** if there are no behavioral outcomes, say so in one short
reason (same spirit as the Plan user rule) and use a single None row rather
than leaving tables blank without explanation.

Do not invent coverage, script names, or outcomes not evidenced by the diff or
repo files.

## Hard rules

- Review/propose only unless the user asks to implement.
- Cap Outcomes, Acceptance, and Regression gap noise; prefer fewer evidenced
  rows.
- Prefer CI/`all-checks` over hooks for expensive checks.
- Not a substitute for Bugbot or Security Review.
