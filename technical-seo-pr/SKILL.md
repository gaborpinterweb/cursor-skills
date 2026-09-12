---
name: technical-seo-pr
description: >-
  Drafts and creates a pull request for technical SEO work (audit fixes + HTML
  reports). Uses a score-first description with improvements, new-content, and
  FROM→TO tables, then a checkbox test plan. Use when the user asks for a
  technical SEO PR, /technical-seo-pr, or a PR after technical-seo-implement /
  a technical-seo-audit re-run. Do not use generic create-pr for this work.
disable-model-invocation: true
---

# Technical SEO PR

Opens a **technical SEO** pull request. Draft first; create only after the user
confirms. Do **not** use `create-pr` for this work.

Requires a **post-change** `technical-seo-audit` so the score is real. If none
exists (or the tree has SEO changes after the latest audit), run
`technical-seo-audit` first or ask.

## Score sentence (verbatim shape)

Fill numbers from the latest audit `meta.json` (`passRate`, `counts.pass`,
`counts.scored`, `counts.fail`). Do not invent or round:

```text
This PR is for technical SEO improvements. Technical SEO score after these changes: {{PASS_RATE}}% ({{PASS}}/{{SCORED}} scored; {{FAIL}} fail). Audit HTML reports included.
```

Example: `This PR is for technical SEO improvements. Technical SEO score after these changes: 95% (69/73 scored; 0 fail). Audit HTML reports included.`

The PR body **starts** with that sentence (nothing before it).

## Workflow

### 1. Resolve site + latest audit

Inspect the **target repository** (git/repo root). Discover website packages the same way as `technical-seo-audit` (`sites/`, `websites/`, `apps/`, workspace globs, or repo-root children — do not assume `sites/`).

- **2+ website packages** → multi-site. Infer `<site-dir>` from the branch name, changed paths, or the user. Latest audit = newest complete folder under `<site-dir>/reports/seo/YYYY-MM-DD-HHmm/`.
- **0 or 1 website package** → single-site. Latest audit = newest complete folder under `reports/seo/YYYY-MM-DD-HHmm/`.

Complete means `index.html` + `meta.json` exist. Read `meta.json` for the score.
If the user names a report folder, use that. Also check the other layout if nothing complete is found.

If no complete post-change audit: stop and say to run `technical-seo-audit`.

### 2. Gather git context (parallel)

Same base-branch rule as `review-branch` (fresher of `main` / `master`, unless
the user names a base):

- `git status`
- `git diff` (staged + unstaged) and `git diff <base>...HEAD`
- `git log <base>..HEAD` (full messages + oneline)
- Branch tracking / ahead-behind

### 3. Include audit HTML reports (required)

The PR **must** contain the HTML reports — not only mention them.

Stage/commit complete `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/` folders for
**this site** (at least the latest; include other complete runs for this site
that are part of this work). Skip incomplete folders (no `index.html`).

`{{REPORT_ROOT}}` is `reports/seo` (single-site) or `<site-dir>/reports/seo` (multi-site).

**Never** add: `public/`, `resources/_gen/`, other sites’ reports, `.env`,
secrets, or unrelated dirty files.

Right under the score sentence, list report `index.html` paths as bullets
(latest first). No extra heading.

### 4. Classify the diff into three tables

Use commits, the diff, `implementation-summary.md`, and `handoff.md`. Do not
invent rows. One concern per row. Number **1, 2, 3…** in each table.

**Technical SEO improvements** — code, templates, config, assets, schema, nav,
canonicals, hreflang, performance, etc. **Not** copy that belongs in the
content tables.

| Column | What to write |
|--------|----------------|
| Number | 1, 2, 3… |
| Change | Short name, usually `RULE-ID` + label (`M-03 Mobile nav`) |
| What changed | Concrete outcome, not a file dump |
| Why it matters | One SEO/UX reason |

**New content (field was empty or missing)** — a field had no value (empty
front matter, missing `og:image`, missing alt, new locale string). Columns:
Number, Field, Where, Added.

**Content change (FROM → TO)** — existing copy rewritten. Columns: Number,
Field, Where, From, To. Quote the real before/after strings from the diff.

If a table has no rows, keep the heading and one row: `—` / `None`.

Escape `|` in cells. Keep From/To faithful; do not paraphrase marketing copy.

### 5. Test plan

Checkbox list derived from **this** diff (not a generic SEO lecture). Always
include a build/check that exists in the site (`hugo`, `npm test`, linkchecker)
when the repo has one. Add locale/page checks for copy and canonicals, plus
any UI the diff touched (mobile nav, footer, 404). No empty `- [ ]` items.

### 6. Title

Short, imperative; include the site name. Example:
`Technical SEO improvements for Opsential`. Match repo PR/commit style.

### 7. Ask for confirmation (required)

Show: base → head, title, full body, files that will be committed (reports +
any still-uncommitted SEO implementation), ticket/preview if found.

Wait. Do **not** commit, push, or `gh pr create` until the user confirms
(e.g. “looks good”, “create it”, “yes”). Accept edits.

### 8. Create (after confirmation only)

1. If needed, commit staged SEO implementation + report folders (message
   focuses on why, not a file list). Never `git add -A` at repo root in a
   monorepo.
2. `git push -u origin HEAD` if needed.
3. `gh pr create --base <base> --title "…" --body` via HEREDOC.
4. Return the PR URL.

If a PR already exists for the branch: give its URL and offer `gh pr edit`
instead of opening a duplicate.

## PR body template

```markdown
This PR is for technical SEO improvements. Technical SEO score after these changes: {{PASS_RATE}}% ({{PASS}}/{{SCORED}} scored; {{FAIL}} fail). Audit HTML reports included.

- `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/index.html`

## Technical SEO improvements

| Number | Change | What changed | Why it matters |
|--------|--------|--------------|----------------|
| 1 | | | |

## New content (field was empty or missing)

| Number | Field | Where | Added |
|--------|-------|-------|-------|
| 1 | | | |

## Content change (FROM → TO)

| Number | Field | Where | From | To |
|--------|-------|-------|------|----|
| 1 | | | | |

## Test plan

- [ ] …
```

## Confirmation reply shape

```markdown
**Branch:** `<head>` → `<base>`

**Title:** …

**Body:**
---
…full draft…
---

**Will commit:** <report paths and any uncommitted SEO files, or none>

Confirm to create, or send edits.
```

## Do not

- Create, commit, or push before confirmation
- Use the generic `create-pr` body (Summary / Changes / Decisions)
- Omit reports from the commit
- Invent scores, FROM/TO copy, or ticket/preview URLs
- Put copy-only edits only in the improvements table (they belong in New
  content or Content change)
