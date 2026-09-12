---
name: technical-seo-implement
description: >-
  Implements technical SEO fixes from a technical-seo-audit handoff.md
  (IMP-* tasks) produced under reports/seo/YYYY-MM-DD-HHmm/ (single-site) or
  <site-dir>/reports/seo/YYYY-MM-DD-HHmm/ (multi-site). Use when the user asks
  to apply SEO audit recommendations, implement scorecard fixes, or follow up
  after a technical-seo-audit HTML report.
---

# Technical SEO Implement

Implements fixes from a **technical-seo-audit** run. Do not re-run the full 69-rule audit from scratch.

## Required input

Resolve the audit folder the same way as `technical-seo-audit` (discover website packages; do not assume `sites/`):

- **Single-site:** `reports/seo/YYYY-MM-DD-HHmm/`
- **Multi-site:** `<site-dir>/reports/seo/YYYY-MM-DD-HHmm/` (infer `<site-dir>` from the user, cwd, or changed paths)

Prefer:

```text
{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/handoff.md
```

Also useful:

- `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/index.html` (context)
- `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/meta.json` (counts / commit)

If the user names a folder, use that. Honor older `audit/`, `seo/*/technical-seo-scorecard.md`, or the other layout if that is where the run actually lives.

If missing, ask for the audit folder path. If they need a new audit first → `technical-seo-audit`.

## Workflow

1. Read `handoff.md` fully.
2. Implement **High** then **Medium** `IMP-*` tasks. Skip **Low** unless the user asks.
3. For each task: satisfy **Acceptance**, match project patterns, keep diffs focused.
4. Preserve intentional `noindex` and notes from the audit.
5. Walk **Common implementation pitfalls** against the diff. Fix anything that applies before calling the work done.
6. When done, write a short summary (and optionally `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/implementation-summary.md`):

```markdown
# SEO implementation summary

| IMP-ID | Rule | Status | Files |
|--------|------|--------|-------|
| IMP-001 | T-02 | done | … |

## Blocked
- …

## Next
Re-run technical-seo-audit (new dated folder) with fresh SF/PSI attachments if available.
```

## Rules

- Do not invent scope beyond `handoff.md` unless the user expands it.
- Ignore SF security-header noise unless an IMP task includes it.
- Prefer shared layouts/partials over one-off page edits.
- No drive-by refactors.

## Common implementation pitfalls

Check these after the IMP tasks. They are how SEO/a11y diffs look finished in HTML but fail review. Skip items that do not exist on this stack.

**CMS vs template**

- An editor hint is not an alt field. If the claim is “images have real alt text,” the CMS schema must expose `alt` (or equivalent) and the template must pass it through. Empty `alt=""` is correct only for decorative images inside a named link.
- Do not claim a CMS-driven fix is done if the field cannot be filled or the layout ignores it.

**i18n**

- Localize every chrome surface you touch (desktop nav, mobile nav, lightbox fallbacks, `aria-label`s). One locale string next to an untranslated sibling is a miss.
- Do not call the site’s i18n helper unless that catalog actually exists. Match the project’s existing locale pattern, or fallbacks always ship the default language.

**Crawlers vs users**

- A real `href` on legal/footer links is not enough if JS still `preventDefault()`s on modal/dialog attributes. Crawlers see the URL; JS users must either navigate or the PR must say the modal is intentional.
- If you special-case some links (modals, absolute URLs, `mailto:`), keep the same URL helper (`relURL` / `absURL` / site-root prefix) on the **default** branch. Bare relative paths resolve wrong on nested routes.

**Error pages**

- Static hosts often have two 404s: the generator’s `layouts/404` (site root) and a self-contained `/errors/404.html` for the CDN. Changing robots or branding on one leaves the other wrong. Prefer `noindex, follow` on both.

**One source of truth**

- Compute title and description once in the base layout; pass them into JSON-LD and OG/Twitter. Duplicated `| default` expressions drift.
- Prefer a data attribute on the menu entry (`modal: privacy`) over string-matching `"#privacy"`. New links will miss hardcoded branches.

**Config leftovers**

- If you change URL mode (`relativeURLs`, `baseURL`, trailing-slash), delete comments and client `<base href>` workarounds that describe the old mode.
- Do not add slash-trim/re-add on `baseURL` when the generator already guarantees the trailing slash.

**A11y chrome**

- `role="dialog"` / `aria-modal="true"` requires moving focus in, trapping Tab, and restoring focus on close. Otherwise do not claim it is a dialog.

**CI / checks**

- If the link checker rebuilds the site (e.g. `--baseURL /`), it must not overwrite the production `public/` (or equivalent) that later pa11y / HTML-validate steps read. Use a separate output directory.
- Do not serialize a check job on the production build artifact if that job runs its own independent build.
- Do not commit bulky audit HTML (`reports/seo/`). Gitignore it. The PR description must match the diff.
