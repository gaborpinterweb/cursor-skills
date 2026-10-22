---
name: explain-changes
description: >-
  Explains the intention and behaviour of diffs for a user-supplied file list
  against origin/main or origin/master. Numbered, editor-navigable entries.
  Not a review: no bugs, style, or compliance checks. Use when the user runs
  /explain-changes or asks to explain branch changes in listed files.
disable-model-invocation: true
---

# Explain Changes

Explain **intention** and **behaviour** of changes in the files the user lists.
Do **not** review for errors, style, security, or AGENTS compliance.

## Input

Requires a **list of files** (paths relative to the repo root, or absolute under
the repo). If the user names none, ask for the list and stop.

Optional: a named base (`against develop`, `vs origin/release/x`). That wins
over the default.

## Base branch

Default base is `origin/main` or `origin/master`, whichever is **fresher**.

1. Prefer remote-tracking refs; else local `main` / `master`.
2. If only one exists, use that.
3. If both exist, pick the tip with the newer committer date:
   `git log -1 --format=%cI <ref>`
4. If neither exists, ask.
5. User-named base wins.

One short line in the reply only, e.g. `Base: origin/main`. Do not explain the
fresher-ref logic.

## Diff scope

For **only** the listed files:

```bash
git merge-base HEAD <base>
git diff <merge-base> -- <file1> <file2> ...
git diff <merge-base>...HEAD -- <file1> <file2> ...
```

Include committed + working-tree changes for those paths unless the user asks
for commits-only or uncommitted-only.

Ignore unrelated files that appear on the branch. If a listed file has no diff
vs base, say so briefly and skip it.

## What to extract

Per file, identify the behavioural units that changed:

- New or substantially changed functions / methods / components
- New call sites or wiring that changes runtime behaviour
- Meaningful control-flow or data-contract changes

Skip pure formatting, renames with no behaviour change, and noise unless the
user asks for everything.

Group by concern when one file mixes features (e.g. only the ecommerce helpers,
not an unrelated consent helper that also landed in the same file). Prefer the
user’s stated ticket/scope when present.

## Output format

1. One-line base note.
2. One or two sentences: overall **intention** of these changes.
3. A **numbered** list. Each item:
   - A markdown link that opens the editor at the definition or primary change:
     `[label](path/to/file.ext:LINE)` (repo-relative path, 1-based line).
   - A short clause on **what it does** / how behaviour changes (not how to
     improve it).
4. Optional second numbered block for **call sites / wiring** if that helps
   navigation; same link style.

Keep entries scannable. One sentence per item is enough unless the behaviour is
inherently multi-step.

### Link rules

- Point at the function/component definition, or the clearest changed line.
- After editing nothing yourself, resolve line numbers from the **current**
  working tree (read or grep), not from memory.
- Do not use code-fence citations as a substitute for the numbered links; the
  list itself must be clickable.

## Hard rules

- **Explain only** — no findings, suggestions, or “consider fixing…”.
- **Listed files only** — do not expand scope to the rest of the branch.
- **Behaviour over implementation trivia** — name what the user/runtime sees
  or what dataLayer/API/UI does, not every line touched.
- If the diff is empty for all listed files, say that and stop.
