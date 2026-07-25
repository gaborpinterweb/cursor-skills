# technical-seo-audit

Audit-only Cursor skill: evaluates a static-site frontend against 60+ technical SEO rules and produces a client-ready HTML report. It does **not** change site code.

## When to use it

Ask Cursor for a technical SEO audit, scorecard, or HTML audit report — especially when you can attach Screaming Frog Issues or PageSpeed exports.

## How it works

1. Creates a new dated folder `audit/YYYY-MM-DD-HHmm/` in the project (never reuses a previous run).
2. Copies any attachments into `attachments/` and snapshots this skill’s `SKILL.md` + `rules.md` into `skill-sources/`.
3. Evaluates every rule in [`rules.md`](rules.md) against the codebase (and attachments), scoring each as pass / partial / fail / n/a / unknown.
4. Builds `index.html` from [`report-template.html`](report-template.html) + [`report.css`](report.css) — pass-rate gauge, status tiles, By priority / By rule views, and collapsible findings.
5. Writes `meta.json` (machine summary) and `handoff.md` (IMP-* tasks for High/Medium fail & partial rules).
6. Stops and points you at `index.html`. Fixes are left to [`technical-seo-implement`](../technical-seo-implement/).

## Outputs

| File | Role |
|---|---|
| `index.html` | Client-facing report (open this) |
| `report.css` | Report styles |
| `meta.json` | Counts, commit, attachments, areas |
| `handoff.md` | Implementation task list |
| `attachments/` | Copied SF/PSI (etc.) originals |
| `skill-sources/` | Verbatim skill + rules used for that run |

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |
| [`rules.md`](rules.md) | Full rule checklist the audit must evaluate |
| [`report-template.html`](report-template.html) | HTML shell / placeholders for the report |
| [`report.css`](report.css) | Shared report styling |
| [`meta.example.json`](meta.example.json) | Shape of `meta.json` |
| [`checklist.md`](checklist.md) | Supporting checklist notes |

## Pairing

After the report: run **technical-seo-implement** against that run’s `handoff.md`, then re-run this audit in a **new** dated folder to measure improvement.
