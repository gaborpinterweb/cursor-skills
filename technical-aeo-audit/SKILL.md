---
name: technical-aeo-audit
description: >-
  Runs a technical Answer Engine Optimization (AEO) audit on a static-site
  frontend, evaluates ~47 rules across 10 checklist areas (plus Answerlint
  attachments), and writes a dated self-contained HTML report under
  reports/aeo/YYYY-MM-DD-HHmm/ (single-site) or
  <site-dir>/reports/aeo/YYYY-MM-DD-HHmm/ (multi-site) with collapsible
  pass/partial/fail results, attachments, and commit hash. Use when the user
  asks for an AEO audit, GEO audit, AI visibility / citation readiness
  scorecard, or Answerlint-based report.
---

# Technical AEO Audit

Audit-only. Do **not** implement fixes. Produce a **self-contained HTML report** plus a handoff file for `technical-aeo-implement`.

## When to run

User asks for a technical AEO / GEO / AI-visibility audit or scorecard, or attaches Answerlint JSON/CSV (or `llms lint` / prompt-pack) exports.

## Inputs

| Input | Required | Notes |
|-------|----------|--------|
| Codebase | Yes | Project root (prefer built HTML / SSG output for JS-heavy sites) |
| Attachments | No | Answerlint audit JSON/CSV, `llms lint` output, optional prompt-pack CSV, schema validator screenshots |
| Site URL | No | Optional context for live fetches — **do not** put in the HTML header |

Copy every user-attached audit artifact into the report folder (see below).

Primary free attachment: [Answerlint](https://www.npmjs.com/package/answerlint) (`npx answerlint audit …`).

## Report location (do this first)

Inspect the **target repository** (the project being audited), not this skills repo. Use the git/repo root. Do **not** assume websites live in `sites/` — find them.

A **website package** is a deployable frontend (own Hugo/Astro/Next/Eleventy/Vite config, `hugo.toml`/`config.toml`, or `content/`+`layouts/`, or a `package.json` that is a site app). Skip libs, APIs, `shared`, `node_modules`, `_template`, `.git`, `reports`.

Find packages in this order; use the first parent that contains **2+** website packages:

1. Common containers at repo root: `sites/`, `websites/`, `apps/`, `web/`, `frontends/`, `packages/`
2. Workspace globs (`pnpm-workspace.yaml`, `package.json` `workspaces`, `turbo.json`, `nx.json`)
3. Immediate repo-root children

Then:

- **2+ website packages** → multi-site. Report root: `<site-dir>/reports/aeo` (e.g. `websites/acme/reports/aeo`, `apps/marketing/reports/aeo`). Infer `<site-dir>` from the user, cwd, or changed/audited paths. If more than one could apply, ask.
- **0 or 1 website package** → single-site. Report root: `reports/aeo` at the repo root. Do **not** nest under the site folder.

Store `site` (slug or `null`), `siteDir` (path or `null`), and `reportRoot` in `meta.json`.

## Output layout

Create a new dated folder under the report root (never reuse a folder name):

```text
{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/
  index.html              # Main report (open this) — self-contained (CSS inlined)
  meta.json               # Machine-readable summary
  handoff.md              # For technical-aeo-implement
  skill-sources/
    SKILL.md              # Verbatim copy used for this run
    rules.md              # Verbatim copy used for this run
  attachments/            # Copied originals
    <original-filename>
```

`{{REPORT_ROOT}}` is `reports/aeo` (single-site) or `<site-dir>/reports/aeo` (multi-site). Example: `reports/aeo/2026-07-24-1512/` or `websites/acme/reports/aeo/2026-07-24-1512/`.

- Always use **local date + time**: `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/` (24h clock, zero-padded).
- If that exact minute folder already exists, append seconds: `{{REPORT_ROOT}}/YYYY-MM-DD-HHmmss/`.
- Resolve **git commit**: `git rev-parse --short HEAD` (and note dirty tree if `git status --porcelain` is non-empty). If not a git repo, set commit to `n/a`.
- Resolve **commit URL** from `git remote get-url origin` (or first remote):
  - **GitHub** (`github.com`): `https://github.com/<owner>/<repo>/commit/<full-sha>`
  - **GitLab** (`gitlab.com` or a host containing `gitlab`): `https://<host>/<namespace>/<repo>/-/commit/<full-sha>`
  - Strip trailing `.git`; support SSH (`git@host:path.git`) and HTTPS remotes
  - Use **full SHA** in the URL; display the **short** hash in the UI (link opens in a new tab: `target="_blank"` `rel="noopener"`)
  - Dirty trees: keep the link on the hash only, append ` (dirty)` as plain text
  - If no recognizable remote: plain text hash (no link)
- Display **date with time and timezone** in the report header. Store the same in `meta.json` `date`. Store `commit`, `commitFull`, `commitUrl` (or `null`), `commitDirty` in `meta.json`.

## Workflow

1. Resolve the report root (above). Create `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/`, `attachments/`, and `skill-sources/`.
2. Copy all provided attachments into `attachments/` (preserve filenames). Record them in `meta.json` and the HTML header as links (`target="_blank"` `rel="noopener"`).
3. Copy this skill’s `SKILL.md` and `rules.md` into `skill-sources/` (verbatim). Embed the same full text behind the clickable **skill** / **rules** words in the method line.
4. Record `model` (Cursor model name) and `analysisScope` (`static site files` / `a live URL` / `static site files and a live URL`).
5. Parse Answerlint JSON/CSV if present; map fail/warn checks into rules (see [rules.md](rules.md)).
6. Evaluate **every rule** in [rules.md](rules.md) (47) against the codebase (+ attachments). Status: `pass` | `partial` | `fail` | `n/a` | `unknown`.
7. Build `index.html` from [report-template.html](report-template.html). Keep the inlined `<style>` block and visual structure identical across runs. Do **not** emit a separate `report.css`.
8. Write `meta.json` and `handoff.md`.
9. Stop. Tell the user the path to `index.html`. Do not implement.

## Status meanings (PageSpeed-like)

| Status | UI label | Meaning |
|--------|----------|---------|
| `fail` | Failed | Broken / missing for important pages |
| `partial` | Partially failed | Present but incomplete or inconsistent |
| `pass` | Passed | Meets the rule |
| `n/a` | N/A | Not applicable (excluded from fail rates) |
| `unknown` | Unknown | Needs Answerlint / live prompt check; treat like non-pass in summary counts separately |

## Priorities

| Priority | When |
|----------|------|
| **High** | AI crawler access, answer-shaped openings, extractable structure, citation readiness, critical Answerlint fails |
| **Medium** | Schema (when applicable), entities, trust, freshness, comparison pages, attachment mapping |
| **Low** | Polish, optional llms.txt lint, soft citation style |

## Report UI requirements

`index.html` **must** include, in order:

1. **Header** — report title **Technical AEO Audit**, **date with time + timezone in parentheses**, **commit hash** linked when possible (+ dirty flag if dirty). Do **not** show site URL, stack, or phase/before/after labels.
2. **Method lines** (above the attachment chips) — two short sentences, in this order:

   > A [handoff.md](handoff.md) was generated so agents can fix the failed and partial issues.

   > This report was generated based on analysis performed by Cursor ({{MODEL}}) using the **skill** and **rules** over {{ANALYSIS_SCOPE}}, and the following attachments:

   - End the second sentence with `attachments:` (colon) so it leads into the attachment chips
   - The words **skill** and **rules** are underlined inline `<label>` controls that open modals with the **full verbatim** `SKILL.md` / `rules.md` (plus “Open file” + Copy)
   - Link `handoff.md` with `target="_blank"` `rel="noopener"`
   - Store `model` and `analysisScope` in `meta.json`
3. **Attachments** — list of files in `attachments/`; each link opens in a **new tab**. If none: show `None provided.`
4. **Summary card** — pass-rate circle + status tiles. `{{COUNT_SCORED}}` = pass + partial + fail + unknown (exclude n/a).
5. **Tabs** — **By priority** (default) / **By rule**.
6. **Overview card** — table for the active tab:
   - **By priority** — High / Medium / Low / Total
   - **By rule** — 10 checklist areas in fixed `rules.md` order (codes **B A H F N T D C V X**); count links scroll to `#area-B` … `#area-X`
   - **Row colors** — red if Failed &gt; 0, yellow if Failed = 0 and Partial &gt; 0, else green (script applies `row-fail` / `row-partial` / `row-pass`)
   - **Count cells** — full cell HTML: `<a class="count …">` or `<span class="count count-zero">0</span>`
   - **By rule names** — keep hardcoded `<label class="method-link" for="modal-area-…">` and area-guide modals
7. **Issue list** — priority view `#prio-high-fail` → … → `#prio-passed`; rule view `#area-B` … `#area-X` with `.area-group` cards
8. Each rule is a **`<details>` collapse** with Why / Evidence / How to fix (optional Answerlint refs)
9. **Method / area pops** — checkbox-driven modals with Copy + Close + backdrop; embeds via `{{EMBED_SKILL_MD}}` / `{{EMBED_RULES_MD}}`

Zero counts: use `<span class="count count-zero">0</span>` or `stat-tile is-zero` — not a link.

Keep styles embedded in `index.html`. When filling embeds, replace only the `{{EMBED_*}}` placeholders inside `<pre class="skill-file-body">` (do not globally replace strings that may appear inside skill docs).

Do not remove the footer script (row colors, scroll-safe modal open, Copy).

## Scoring (headline)

`passed / (passed + partial + fail + unknown)` as a percentage (exclude `n/a`).  
Gauge: &lt;50 poor (red), 50–89 average (orange), ≥90 good (green).

## Handoff (`handoff.md`)

Generate implementation tasks only for `fail` and `partial` rules with priority High then Medium (Low only if user asked for full polish):

```markdown
# AEO implement handoff — YYYY-MM-DD

Commit: <hash>
Report: {{REPORT_ROOT}}/YYYY-MM-DD-HHmm/index.html

## Tasks
### IMP-001 — <rule id> — <title>
- Priority: High|Medium|Low
- Status: fail|partial
- Why: …
- Scope: …
- Acceptance: …
- Evidence: …
```

## Quality bar

- Evaluate **all** rules in [rules.md](rules.md) (do not stop at 10 areas).
- Every non-`pass` / non-`n/a` rule has evidence (path, Answerlint check name, or “not found in repo”).
- Prefer `n/a` for FAQ/HowTo and comparison rules when page intent does not match.
- Do not treat missing `llms.txt` as an automatic High fail unless the user requested LLM roadmap work.
- Attachments that were provided appear in the header and open in a new tab.
- No code fixes in this skill.
