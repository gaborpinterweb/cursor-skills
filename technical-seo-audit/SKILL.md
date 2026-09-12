---
name: technical-seo-audit
description: >-
  Runs a technical SEO audit on a static-site frontend, evaluates 60+ rules
  (15 checklist areas + Screaming Frog Issues), and writes a dated HTML report
  under reports/seo/YYYY-MM-DD-HHmm/ (single-site) or
  <site-dir>/reports/seo/YYYY-MM-DD-HHmm/ (multi-site) with collapsible
  pass/partial/fail results, attachments, and commit hash. Use when the user
  asks for a technical SEO audit, SEO scorecard, or audit report.
---

# Technical SEO Audit

Audit-only. Do **not** implement fixes. Produce a **self-contained HTML report** plus a handoff file for `technical-seo-implement`.

## When to run

User asks for a technical SEO audit / scorecard / HTML audit report, or attaches Screaming Frog Issues / PageSpeed (PSI) exports.

## Inputs

| Input | Required | Notes |
|-------|----------|--------|
| Codebase | Yes | Project root |
| Attachments | No | SF Issues CSV, PSI HTML/JSON/PDF/screenshots, Unlighthouse export, etc. |
| Site URL | No | Optional context for host/canonical checks — **do not** put in the HTML header |

Copy every user-attached audit artifact into the report folder (see below).

## Report location (do this first)

Inspect the **target repository** (the project being audited), not this skills repo. Use the git/repo root. Do **not** assume websites live in `sites/` — find them.

A **website package** is a deployable frontend (own Hugo/Astro/Next/Eleventy/Vite config, `hugo.toml`/`config.toml`, or `content/`+`layouts/`, or a `package.json` that is a site app). Skip libs, APIs, `shared`, `node_modules`, `_template`, `.git`, `reports`.

Find packages in this order; use the first parent that contains **2+** website packages:

1. Common containers at repo root: `sites/`, `websites/`, `apps/`, `web/`, `frontends/`, `packages/`
2. Workspace globs (`pnpm-workspace.yaml`, `package.json` `workspaces`, `turbo.json`, `nx.json`)
3. Immediate repo-root children

Then:

- **2+ website packages** → multi-site. Report root: `<site-dir>/reports/seo` (e.g. `websites/acme/reports/seo`, `apps/marketing/reports/seo`). Infer `<site-dir>` from the user, cwd, or changed/audited paths. If more than one could apply, ask.
- **0 or 1 website package** → single-site. Report root: `reports/seo` at the repo root. Do **not** nest under the site folder.

Store `site` (slug or `null`), `siteDir` (path or `null`), and `reportRoot` in `meta.json`.

## Output layout

Create a new dated folder under the report root (never reuse a folder name):

```text
{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/
  index.html              # Main report (open this) — self-contained (CSS inlined)
  meta.json               # Machine-readable summary
  handoff.md              # For technical-seo-implement
  skill-sources/
    SKILL.md              # Verbatim copy used for this run
    rules.md              # Verbatim copy used for this run
  attachments/            # Copied originals
    <original-filename>
```

`{{REPORT_ROOT}}` is `reports/seo` (single-site) or `<site-dir>/reports/seo` (multi-site). Example: `reports/seo/2026-07-24-1512/` or `websites/acme/reports/seo/2026-07-24-1512/`.

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
- Display **date with time and timezone** in the report header (e.g. `2026-07-24 15:16 (CEST)` or `2026-07-24 15:16 (UTC+2)`). Store the same in `meta.json` `date`. Store `commit`, `commitFull`, `commitUrl` (or `null`), `commitDirty` in `meta.json`.

## Workflow

1. Resolve the report root (above). Create `{{REPORT_ROOT}}/YYYY-MM-DD-HHmm/`, `attachments/`, and `skill-sources/`.
2. Copy all provided attachments into `attachments/` (preserve filenames). Record them in `meta.json` and the HTML header as links (`target="_blank"` `rel="noopener"`).
3. Copy this skill’s `SKILL.md` and `rules.md` into `skill-sources/` (verbatim). Embed the same full text behind the clickable **skill** / **rules** words in the method line.
4. Record `model` (Cursor model name) and `analysisScope` (`static site files` / `a live URL` / `static site files and a live URL`).
5. Parse SF Issues CSV if present; map rows into rules (see [rules.md](rules.md)).
6. Evaluate **every rule** in [rules.md](rules.md) (≥60) against the codebase (+ attachments). Status: `pass` | `partial` | `fail` | `n/a` | `unknown`.
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
| `unknown` | Unknown | Needs live crawl / PSI; treat like non-pass in summary counts separately |

## Priorities

| Priority | When |
|----------|------|
| **High** | Indexing, core on-page, canonicals, critical SF errors |
| **Medium** | Important UX/SEO quality, CWV readiness, structured data, OG |
| **Low** | Polish, soft length opportunities, security-header noise unless user asked |

## Report UI requirements

`index.html` **must** include, in order:

1. **Header** — report title, **date with time + timezone in parentheses**, **commit hash** linked to GitHub/GitLab commit when possible (+ dirty flag if dirty). Do **not** show site URL, stack, or phase/before/after labels.
2. **Method lines** (above the attachment chips) — two short sentences, in this order:

   > A [handoff.md](handoff.md) was generated so agents can fix the failed and partial issues.

   > This report was generated based on analysis performed by Cursor ({{MODEL}}) using the **skill** and **rules** over {{ANALYSIS_SCOPE}}, and the following attachments:

   - End the second sentence with `attachments:` (colon) so it leads into the attachment chips
   - `{{MODEL}}` — the Cursor model that ran the audit (e.g. `Composer`)
   - `{{ANALYSIS_SCOPE}}` — one of: `static site files` · `a live URL` · `static site files and a live URL` (adjust wording only if needed for clarity)
   - The words **skill** and **rules** are underlined inline `<label>` controls that open modals with the **full verbatim** `SKILL.md` / `rules.md` (plus “Open file”)
   - Link `handoff.md` with `target="_blank"` `rel="noopener"`
   - Store `model` and `analysisScope` in `meta.json`
3. **Attachments** — list of files in `attachments/` immediately under the method lines; each link opens in a **new tab**. If none: show `None provided.`
4. **Summary card** — pass-rate circle + accumulated status tiles only (no tables). Under the circle, a two-line label: `{{COUNT_PASS}} succ. / {{COUNT_SCORED}} total` then `{{PASS_RATE}}% pass rate`. `{{COUNT_SCORED}}` = pass + partial + fail + unknown (exclude n/a).
5. **Tabs** — **By priority** (default) / **By rule**. Switching the tab changes both the Overview card and the issue list sort order (emit **both** lists; show/hide with the same CSS radio tabs).
6. **Overview card** — table for the active tab:
   - **By priority** — High / Medium / Low / Total; count links scroll to `#prio-…` sections
   - **By rule** — 15 checklist areas in fixed `rules.md` order (+ Attachment-driven / SF extras); count links scroll to `#area-T` … `#area-X`
   - **Row colors** — each tbody row (including Total) gets a status background via the template script: **red** if Failed &gt; 0, **yellow** if Failed = 0 and Partial &gt; 0, else **green**. Do not remove the footer `<script>` that applies `row-fail` / `row-partial` / `row-pass`.
   - **Count cells** — replace each `{{H_FAIL}}`, `{{AREA_T_FAIL}}`, `{{COUNT_FAIL_CELL}}`, etc. with a full cell value: either `<a class="count count-…" href="…">N</a>` or `<span class="count count-zero">0</span>` (never a bare number).
   - **By rule names** — keep the hardcoded `<label class="method-link" for="modal-area-…">` controls and the matching area-guide modals (Why it matters / How to fix). Do not replace area names with plain text or strip those modals.
7. **Issue list** (sorted to match the active tab; all rules **collapsed by default**):
   - Priority view: `#prio-high-fail` → … → `#prio-passed`
   - Rule view: `#area-T` … `#area-B` → `#area-X`. Each area is an `.area-group` card; fail/partial rules and the nested “Passed, N/A & unknown” toggle sit inside `.area-body` (indented with a dotted vertical rail) so they read as children of the area title.
8. Each rule is a **`<details>` collapse**:
   - Summary line: status badge + priority badge + rule-id badge + short title (same row)
   - Body: **Why it matters**, **Evidence**, **How to fix**, optional **SF / PSI refs** — expanded body uses a distinct background
9. **Method / area pops** — **skill** / **rules** and By-rule **area names** are inline `<label>`s (never `<details>` in the sentence) that open checkbox-driven modals with Copy + Close + backdrop. Keep the first method sentence as a **single HTML line**. Area guides are static (hardcoded in the template); skill/rules modals embed verbatim content via `{{EMBED_SKILL_MD}}` / `{{EMBED_RULES_MD}}`.

Zero counts: use `<span class="count count-zero">0</span>` or `stat-tile is-zero` — not a link.

Use only the CSS classes already defined in the template’s inlined `<style>` block — unified design across audits. Keep styles embedded in `index.html` so the report renders correctly when opened alone (without `attachments/` or other sibling files).

When filling `{{EMBED_SKILL_MD}}` / `{{EMBED_RULES_MD}}`, HTML-escape the full file text into the modal `<pre class="skill-file-body">` blocks (replace those exact placeholders only — do not globally replace bare `{{SKILL_MD}}` / `{{RULES_MD}}` strings that may appear inside the skill docs). Also copy the files to `skill-sources/SKILL.md` and `skill-sources/rules.md`. Store `site`, `siteDir`, `reportRoot`, `folder`, `skillName`, `model`, `analysisScope`, and `byArea` in `meta.json`.

Each modal header includes **Copy** (clipboard) and **Close**. Keep those controls; do not remove the footer script that colors overview rows, opens modals without scrolling, and handles Copy.

## Scoring (headline)

Show a PageSpeed-style **pass-rate circle**:  
`passed / (passed + partial + fail + unknown)` as a percentage (exclude `n/a`).  
Color the gauge: &lt;50 poor (red), 50–89 average (orange), ≥90 good (green).  
Also show counts — do not invent Semrush-style “Site Health” branding.

## Handoff (`handoff.md`)

Generate implementation tasks only for `fail` and `partial` rules with priority High then Medium (Low only if user asked for full polish):

```markdown
# SEO implement handoff — YYYY-MM-DD

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

- Evaluate **all** rules in [rules.md](rules.md) (do not stop at 15).
- Every non-`pass` / non-`n/a` rule has evidence (path, SF issue name + URL count, or “not found in repo”).
- Intentional `noindex` pages are not automatic High fails — call out intent under Evidence.
- Attachments that were provided appear in the header and open in a new tab.
- No code fixes in this skill.
