# technical-seo-implement

Cursor skill that applies fixes from a **technical-seo-audit** handoff. It implements code changes; it does **not** re-run the full 60+ rule audit from scratch.

## When to use it

After an audit, ask Cursor to implement the SEO handoff / apply audit recommendations / fix High and Medium IMP tasks.

## How it works

1. Reads `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/handoff.md` (ask for the path if missing). `{{REPORT_ROOT}}` is `reports/seo` (single-site) or `<site-dir>/reports/seo` (multi-site).
2. Uses `index.html` / `meta.json` only for context when helpful.
3. Implements **High**, then **Medium** `IMP-*` tasks. Skips **Low** unless you ask for full polish.
4. For each task: meets **Acceptance**, follows existing project patterns, keeps the diff focused.
5. Preserves intentional `noindex` and other audit notes.
6. Finishes with a short summary (optionally `implementation-summary.md` in the audit folder) and suggests a fresh **technical-seo-audit** re-run.

## Required input

```text
{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/handoff.md
```

`{{REPORT_ROOT}}` is `reports/seo` (single-site) or `<site-dir>/reports/seo` (multi-site).

Also useful: that folder’s `index.html` and `meta.json`.

If you still have an older `audit/` or `seo/*/technical-seo-scorecard.md` layout, this skill can honor it.

## Guardrails

- Do not invent scope beyond `handoff.md` unless you expand it in the prompt.
- Ignore Screaming Frog security-header noise unless an IMP task includes it.
- Prefer shared layouts/partials over one-off page edits.
- No drive-by refactors.

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Produced by [`technical-seo-audit`](../technical-seo-audit/). After implementing, re-audit in a new dated folder (ideally with fresh SF/PSI attachments), then open the PR with [`technical-seo-pr`](../technical-seo-pr/).
