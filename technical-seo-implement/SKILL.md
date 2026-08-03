---
name: technical-seo-implement
description: >-
  Implements technical SEO fixes from a technical-seo-audit handoff.md
  (IMP-* tasks) produced under reports/seo/YYYY-MM-DD-HHmm/. Use when the user asks to
  apply SEO audit recommendations, implement scorecard fixes, or follow up
  after a technical-seo-audit HTML report.
---

# Technical SEO Implement

Implements fixes from a **technical-seo-audit** run. Do not re-run the full 69-rule audit from scratch.

## Required input

Prefer:

```text
reports/seo/YYYY-MM-DD-HHmm/handoff.md
```

Also useful:

- `reports/seo/YYYY-MM-DD-HHmm/index.html` (context)
- `reports/seo/YYYY-MM-DD-HHmm/meta.json` (counts / commit)

If the user points at an older `seo/*/technical-seo-scorecard.md`, still honor it, but prefer the new `reports/seo/` layout.

If missing, ask for the audit folder path. If they need a new audit first → `technical-seo-audit`.

## Workflow

1. Read `handoff.md` fully.
2. Implement **High** then **Medium** `IMP-*` tasks. Skip **Low** unless the user asks.
3. For each task: satisfy **Acceptance**, match project patterns, keep diffs focused.
4. Preserve intentional `noindex` and notes from the audit.
5. When done, write a short summary (and optionally `reports/seo/YYYY-MM-DD-HHmm/implementation-summary.md`):

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
