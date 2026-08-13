---
name: review-branch
description: >-
  Pattern/AGENTS compliance review of the current branch against main or master
  (whichever is fresher). Checks regressions, globals, reuse, naming, magic
  values, dead code, and diff-scoped a11y/i18n/CI/secrets. Use when the user
  asks to review a branch, run /review-branch, or wants a pre-merge branch
  review. Not a substitute for Bugbot or Security Review.
disable-model-invocation: true
---

# Review Branch

**Pattern / AGENTS compliance review** — not bug hunting and not a security audit.
Use Bugbot for defects, Security Review for vulns. This skill grades the diff
against repo conventions and `AGENTS.md` / `README.md`.

Review-only. Do **not** fix findings unless the user explicitly asks.

## Base branch

Default base is `main` or `master`, whichever is **fresher**.

1. Prefer remote-tracking refs (`origin/main`, `origin/master`); else local.
2. If only one exists, use that.
3. If both exist, pick the tip with the newer committer date:
   `git log -1 --format=%cI <ref>`
4. If neither exists, use `git symbolic-ref refs/remotes/origin/HEAD` or ask.
5. User-named base wins (`against develop`, `vs release/x`).

Diff scope: merge-base of HEAD and base through working tree (committed + staged
+ unstaged), unless the user asks for commits-only or uncommitted-only.

```bash
git merge-base HEAD <base>
git diff --stat <merge-base>...HEAD
git diff <merge-base>
git status --porcelain
```

**User-facing base note:** one short line only, e.g. `Base: origin/main`.
Do not explain the fresher-ref logic in the reply.

## Before reviewing

1. Read repository `README.md` (if present).
2. Read repository `AGENTS.md` (and nested `AGENTS.md` / `.cursor/rules` for
   **changed paths only**).
3. Skim **siblings and call sites of changed files** for existing patterns
   (partials/components, tokens/variables) — do not audit the whole repo.

## Review guidelines

Evaluate the **diff** against:

- No regressions are introduced
- No global changes introduced
- Components, partials and functions are reused whenever possible
- Naming conventions are followed
- No magic numbers, variables and colors are introduced
- Repository-level patterns are followed
- No dead code introduced
- Repository README.md and AGENTS.md are read and respected

### Regression (static sites: Hugo / Astro)

A **regression** is a change that breaks or degrades existing page behavior or
contracts without clear intent. In Hugo/Astro diffs, look for:

- Shared layout/partial/component edits that alter markup, classes, slots, or
  props used by untouched pages
- Broken includes/imports (`partial`, `partialCached`, Astro component paths)
- Removed/renamed content collections, taxonomies, permalinks, or `url` /
  `slug` behavior that orphan links
- CSS specificity or variable overrides that change styling outside the
  intended page/section
- Asset pipeline regressions (Hugo Pipes / Astro assets): missing fingerprint,
  wrong path, broken `srcset`, lost lazy/eager loading
- i18n: missing locale copy, broken `hreflang`, locale layout drift when only
  one language was meant to change

### Global change (static sites: Hugo / Astro)

A **global change** is anything whose blast radius is site-wide or
cross-section, not confined to the feature’s templates/styles. Flag when the
diff touches:

- Base layouts (`baseof`, root `Layout.astro`, site shell/nav/footer)
- Shared partials/components used across many routes
- Design tokens / `:root` / SCSS variables / theme files
- Global CSS entrypoints or resets
- `hugo.toml` / `astro.config` / build integrations / middleware
- Shared shortcodes, content helpers, or utility modules imported broadly

Local page templates, page-scoped CSS modules/scoped styles, and feature-only
content files are **not** global by default.

Intentional globals are allowed only when clearly required; put them in
**Watchouts** if intentional but eyebrow-raising, **Critical** if accidental or
undocumented vs AGENTS/README.

### Diff-scoped extras

Only check these when the **diff** clearly touches that surface (do not scan
unrelated files):

- Accessibility — if markup/components/templates change
- i18n — if locale files, hreflang, or translated templates change
- Build/CI — if config, deps, pipes/assets, or workflows change
- Secrets / CDN / GDPR / unsafe JS — if scripts, external URLs, or asset hosting change
- Coverage gaps — if shared UI changed with no test or verification note in the
  diff/PR context you have
- Blast radius — if shared layouts/partials/tokens/config changed

## Evidence (required)

Every table row **must** cite evidence:

- **Location**: `path:line` (or `path`) from the diff
- **Reference**: existing pattern, AGENTS/README rule, or sibling file the change
  conflicts with or should have reused — e.g. `AGENTS.md § CSS`, 
  `layouts/partials/image.html`, `src/styles/tokens.css`

No evidence → do not list the finding.

## Workflow

1. Resolve base; print one short `Base: …` line.
2. Collect the diff vs merge-base (include dirty tree unless told otherwise).
3. Read README / AGENTS / local patterns for **touched** areas only.
4. Review changed files against the guidelines.
5. Reply with **only** `Base:` + the two tables. No essays, no fix patches
   unless asked.

## Response format

```text
Base: <ref>
```

### Critical errors

Must-fix before merge. Cap: **8 rows**. Merge duplicates. Prefer highest
severity. Number rows **1.1**, **1.2**, … in priority order. If none:
`| — | — | None | — | — | — |`.

| ID | Location | Issue | Guideline | Reference | Recommended fix |
|----|----------|-------|-----------|-----------|-----------------|
| 1.1 | `path:line` | Concrete problem | Which guideline | AGENTS/README section or existing file/pattern | Short concrete action (not a full patch) |

### Watchouts

Non-obvious changes that may raise eyebrows from leads or fellow developers.
Cap: **8 rows**. Number rows **2.1**, **2.2**, … in priority order. If none:
`| — | — | None | — | — | — |`.

| ID | Location | Change | Why it stands out | Reference | Recommended fix |
|----|----------|--------|-------------------|-----------|-----------------|
| 2.1 | `path:line` | What changed (short) | Why a lead/dev might question it | Related file, pattern, or AGENTS note | Short suggestion, or `Confirm with lead` / `Document in PR` |

**Recommended fix:** one line max. Point at the existing pattern to reuse when possible.
Do not paste multi-line patches here — still review-only until the user asks to fix
(e.g. `fix 1.2`).

## Severity rule

- **Critical errors**: evidenced guideline violations — regressions, accidental
  globals, magic values/colors, dead code, diff-scoped a11y/i18n/CI/security
  breaks, or AGENTS/README contradictions.
- **Watchouts**: evidenced judgment calls — intentional-but-surprising globals,
  partial reuse, approach questions, scope/blast-radius, missing verification.

Do not invent issues. Prefer fewer, evidenced rows over speculative noise.
