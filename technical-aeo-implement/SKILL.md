---
name: technical-aeo-implement
description: >-
  Implements technical AEO fixes from a technical-aeo-audit handoff.md
  (IMP-* tasks) produced under reports/aeo/YYYY-MM-DD-HHmm/ (single-site) or
  <site-dir>/reports/aeo/YYYY-MM-DD-HHmm/ (multi-site). Use when the user
  asks to apply AEO/GEO audit recommendations, implement AI-visibility
  scorecard fixes, or follow up after a technical-aeo-audit HTML report.
---

# Technical AEO Implement

Implements fixes from a **technical-aeo-audit** run. Do not re-run the full 47-rule audit from scratch.

## Required input

Resolve the audit folder the same way as `technical-aeo-audit` (discover website packages; do not assume `sites/`):

- **Single-site:** `reports/aeo/YYYY-MM-DD-HHmm/`
- **Multi-site:** `<site-dir>/reports/aeo/YYYY-MM-DD-HHmm/` (infer `<site-dir>` from the user, cwd, or changed paths)

Prefer:

```text
{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/handoff.md
```

Also useful:

- `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/index.html` (context)
- `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/meta.json` (counts / commit)
- `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/attachments/` (Answerlint JSON/CSV context)

If the user names a folder, use that. Honor older `aeo-audit/` or the other layout if that is where the run actually lives.

If missing, ask for the audit folder path. If they need a new audit first → `technical-aeo-audit`.

## Workflow

1. Read `handoff.md` fully.
2. Implement **High** then **Medium** `IMP-*` tasks. Skip **Low** unless the user asks.
3. For each task: satisfy **Acceptance**, match project patterns, keep diffs focused.
4. Preserve intentional choices noted in the audit (e.g. training-bot disallow vs search-bot allow, `n/a` FAQ/comparison pages).
5. Prefer shared layouts/partials, JSON-LD in base templates, and content patterns that stay Answerlint-friendly (direct answers, schema matching visible FAQ, entity consistency).
6. When done, write a short summary (and optionally `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/implementation-summary.md`):

```markdown
# AEO implementation summary

| IMP-ID | Rule | Status | Files |
|--------|------|--------|-------|
| IMP-001 | A-01 | done | … |

## Blocked
- …

## Next
Re-run technical-aeo-audit (new dated folder) with fresh Answerlint JSON/CSV if available.
```

## Rules

- Do not invent scope beyond `handoff.md` unless the user expands it.
- Do not treat missing `llms.txt` as mandatory unless an IMP task includes it.
- Do not add FAQPage/HowTo JSON-LD that does not match visible content.
- Prefer shared layouts/partials over one-off page edits.
- No drive-by refactors.
- Do not re-litigate SEO-only work already covered by `technical-seo-implement` unless the IMP task is AEO-specific.
