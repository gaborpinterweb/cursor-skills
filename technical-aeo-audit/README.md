# technical-aeo-audit

Audit-only Cursor skill: evaluates a static-site frontend against ~47 technical AEO rules (10 checklist areas + Answerlint attachments) and produces a client-ready HTML report. It does **not** change site code.

## When to use it

Ask Cursor for a technical AEO audit, GEO / AI-visibility scorecard, or HTML audit report — especially when you can attach [Answerlint](https://www.npmjs.com/package/answerlint) JSON/CSV exports.

## How it works

1. Creates a new dated folder `aeo-audit/YYYY-MM-DD-HHmm/` in the project (never reuses a previous run).
2. Copies any attachments into `attachments/` and snapshots this skill’s `SKILL.md` + `rules.md` into `skill-sources/`.
3. Evaluates every rule in [`rules.md`](rules.md) against the codebase (and attachments), scoring each as pass / partial / fail / n/a / unknown.
4. Builds a self-contained `index.html` from [`report-template.html`](report-template.html) (CSS inlined) — pass-rate gauge, status tiles, By priority / By rule views, and collapsible findings.
5. Writes `meta.json` (machine summary) and `handoff.md` (IMP-* tasks for High/Medium fail & partial rules).
6. Stops and points you at `index.html`. Fixes are left to [`technical-aeo-implement`](../technical-aeo-implement/).

## Outputs

| File | Role |
|---|---|
| `index.html` | Client-facing report (open this; styles embedded) |
| `meta.json` | Counts, commit, attachments, areas |
| `handoff.md` | Implementation task list |
| `attachments/` | Copied Answerlint (etc.) originals |
| `skill-sources/` | Verbatim skill + rules used for that run |

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |
| [`rules.md`](rules.md) | Full rule checklist the audit must evaluate |
| [`report-template.html`](report-template.html) | Self-contained HTML shell (inlined CSS + placeholders) |
| [`meta.example.json`](meta.example.json) | Shape of `meta.json` |
| [`checklist.md`](checklist.md) | Supporting checklist notes |
| [`examples/mock-report.html`](examples/mock-report.html) | Sample report for UI review |

## Pairing

After the report: run **technical-aeo-implement** against that run’s `handoff.md`, then re-run this audit in a **new** dated folder to measure improvement.