# technical-seo-pr

Cursor skill that **drafts** a technical-SEO pull request (score sentence, three
tables, checkbox test plan), includes the audit HTML reports, asks for
confirmation, then creates the PR with `gh`.

It does **not** open a PR until you approve the message.

## When to use it

Ask Cursor to open a technical SEO PR, or run `/technical-seo-pr`, after
`technical-seo-implement` and a fresh `technical-seo-audit`.

Do **not** use [`create-pr`](../create-pr/) for this work — the body shape is
different.

## How it works

1. Detects single-site vs multi-site (discovers website folders; does not assume `sites/`), then reads the latest complete `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/meta.json` (`reports/seo` or `<site-dir>/reports/seo`) for the pass rate.
2. Makes sure that site’s audit folders (`index.html` and the rest of the run)
   are part of the PR.
3. Classifies the diff into:
   - **Technical SEO improvements** — Number, Change, What changed, Why it matters
   - **New content (field was empty or missing)**
   - **Content change (FROM → TO)**
4. Adds a checkbox **Test plan** from the actual changes.
5. Shows the draft and waits for confirmation.
6. On approval: commits remaining SEO + report files if needed, pushes, runs
   `gh pr create`, returns the PR URL.

## Opening sentence

```text
This PR is for technical SEO improvements. Technical SEO score after these changes: 95% (69/73 scored; 0 fail). Audit HTML reports included.
```

Numbers come from the latest audit (`passRate`, pass / scored, fail count).

## Confirmation gate

No invented scores or FROM/TO copy. Reports must be in the commit, not only
linked in prose.

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

After [`technical-seo-implement`](../technical-seo-implement/) and a new
[`technical-seo-audit`](../technical-seo-audit/) run. Optional:
[`review-branch`](../review-branch/) first if you want a compliance pass.
