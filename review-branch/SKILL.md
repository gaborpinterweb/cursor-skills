---
name: review-branch
description: >-
  Pattern/AGENTS compliance review of the current branch against main or master
  (whichever is fresher). Checks regressions (including silent CSS cascade /
  specificity / overflow), globals, reuse, naming, magic values, dead code,
  diff-scoped a11y/i18n/CI/secrets, and emits manual regression test steps when
  UI CSS/markup moved. Use when the user asks to review a branch, run
  /review-branch, or wants a pre-merge branch review. Not a substitute for
  Bugbot or Security Review.
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
4. When CSS/SCSS moved or selectors were re-scoped, also skim **unchanged
   consumers** that may still win the cascade (legacy module rules, prose
   `a`/`p`, shared hosts for overlays/embeds) — cite them as Reference even if
   they are outside the diff hunk.

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
- **Silent CSS regressions** (see below) when selectors move or shared styles lift
- Asset pipeline regressions (Hugo Pipes / Astro assets): missing fingerprint,
  wrong path, broken `srcset`, lost lazy/eager loading
- i18n: missing locale copy, broken `hreflang`, locale layout drift when only
  one language was meant to change

#### Silent CSS regressions

Trigger when the diff **moves, flattens, re-scopes, or re-orders** selectors
(out of a parent block, into a shared partial/component, SCSS `@use` order
change, new `:has` / attribute / state selectors, absolute + `overflow` /
`aspect-ratio` on embeds or players).

Check (cite changed file **and** the competing consumer or host when relevant):

| Check | What to look for |
|-------|------------------|
| **Scope lift** | Rules left a stronger parent (e.g. `.article__body`, `.block__x`) without raising specificity or excluding legacy consumers — treat as a default Critical/Watchout cue |
| **Specificity drop** | Nested → fewer classes/elements; compare to **unchanged** winners (prose `a`/`p`, legacy embed rules) |
| **Cascade / load order** | Equal specificity + later entrypoint file wins |
| **State selectors** | Rules that only apply after attribute/state changes (e.g. `[open]`, `.is-active`, media `src` set, dialog open) |
| **Layout overflow** | `position: absolute` + fixed `aspect-ratio` + `overflow: hidden` vs content that can grow (overlays, banners, consent UI) |
| **Theme / surface AA** | Link or text tokens on card/tint/overlay surfaces (not only body); light vs dark when themes exist |

A **scope-lift + competing consumer path** is enough evidence for a table row.
You do not need a browser measurement to list it; put confirmation in
**Manual regression checks**.

Severity: restyles untouched pages or fails AA / makes a control untappable →
**Critical**. Clip only on a rare state/viewport but content still reachable →
**Watchout** (or Critical if CTA/link is unusable).

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

- Accessibility — if markup/components/templates change (include contrast on
  non-default surfaces when colors or overlay/card UI change)
- i18n — if locale files, hreflang, or translated templates change
- Build/CI — if config, deps, pipes/assets, or workflows change
- Secrets / CDN / GDPR / unsafe JS — if scripts, external URLs, or asset hosting
  change; for consent-gated third-party UI also note the **denied / blocked**
  path must stay usable
- Coverage gaps — if shared UI changed with no test or verification note in the
  diff/PR context you have
- Blast radius — if shared layouts/partials/tokens/config changed

## Evidence (required)

Every table row **must** cite evidence:

- **Location**: `path:line` (or `path`) from the diff
- **Reference**: existing pattern, AGENTS/README rule, sibling file, or
  **unchanged consumer** the change conflicts with — e.g. `AGENTS.md § CSS`,
  a shared partial/component path, a legacy stylesheet that still targets the
  same elements

No evidence → do not list the finding.

For silent CSS: a diff hunk showing a scope lift **plus** a Reference to the
competing selector/file is sufficient. Do not drop the row only because it is
unverified in a browser.

## Workflow

1. Resolve base; print one short `Base: …` line.
2. Collect the diff vs merge-base (include dirty tree unless told otherwise).
3. Read README / AGENTS / local patterns for **touched** areas only; for CSS
   scope lifts, skim competing consumers as above.
4. Review changed files against the guidelines.
5. Reply with **only** `Base:` + the two tables + **Manual regression checks**
   when required (see below). No essays, no fix patches unless asked.

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

### Manual regression checks

Emit this section when the diff changes **UI CSS/SCSS, shared markup for
components/partials, or interactive overlay/embed behavior** — or when any
table row cites a silent CSS / layout / AA issue. Skip with one line
`### Manual regression checks` / `None (no UI CSS or interactive markup in diff)`
when not applicable.

Cap: **8 steps**. Each step is one line and must include:

- **Where** — route or page type from this repo (derive from the diff; do not
  invent product URLs)
- **Viewport** — e.g. ~375px and a desktop width
- **Theme** — light and/or dark when the project has themes and color/surface matters
- **Action / state** — load, open/close, before/after a state attribute, denied consent, etc.
- **Pass look** — what “good” is (no unexpected margin/clip, AA contrast, control tappable, box does not shrink below prior size)

Prefer **prod or `main` vs preview/local** wording when a shared style moved.
When consent or third-party gating exists in the diff, include at least one
**denied** and one **after-grant** step. For absolute + overflow hosts, include
a hit-target check at the clipped edge when relevant.

Example shape (adapt to **this** repo’s routes and components; do not copy
product-specific paths):

```text
### Manual regression checks
1. Affected page type — 375 + desktop — light + dark — after interactive state that applies new selectors — no unexpected margin/clip vs main/prod.
2. Same — light — overlay/card link contrast AA ≥ 4.5:1; notice/body copy margins unchanged vs main.
3. Host that mounts an overlay in a fixed-ratio box — 375 — denied/blocked path — full overlay visible; edge control tappable.
4. Same — desktop — after open — host keeps prior ratio/size (grows only if content requires it).
```

## Severity rule

- **Critical errors**: evidenced guideline violations — regressions (including
  silent CSS), accidental globals, magic values/colors, dead code, diff-scoped
  a11y/i18n/CI/security breaks, or AGENTS/README contradictions.
- **Watchouts**: evidenced judgment calls — intentional-but-surprising globals,
  partial reuse, approach questions, scope/blast-radius, missing verification,
  scope lifts that look intentional but lack a consumer audit note.

Do not invent issues without a diff or consumer Reference. Prefer fewer,
evidenced rows over speculative noise — but a **scope lift with a named
competing selector** is not speculative; list it and confirm via Manual checks.
