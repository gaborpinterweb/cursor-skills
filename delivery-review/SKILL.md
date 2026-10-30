---
name: delivery-review
description: >-
  Delivery overview of the open PR for the current (or named) branch: code
  review (branch/AGENTS compliance, including silent CSS regressions), PR
  review (description and feedback vs diff), and Linear ticket review vs
  implementation. Emits three concise tables (C# / P# / T#) plus Manual
  regression checks when UI CSS/overlays moved. Use when the user runs
  /delivery-review or /review-pr, asks for a delivery review, or wants a
  pre-merge overview.
disable-model-invocation: true
---

# Delivery Review

Review-only delivery overview. Do **not** fix code, edit the PR, or merge.
Prefer `gh` for GitHub. Goal: a scannable overview; user can ask for details by
row number (e.g. `C2`, `P1`).

## Workflow

### 1. Sync gate

Resolve branch: user-named, else current (`git branch --show-current`).

```bash
git fetch origin <branch> 2>/dev/null || true
git rev-parse HEAD
git rev-parse @{u} 2>/dev/null || git rev-parse origin/<branch>
git status -sb
```

If local HEAD and remote tip **differ**: explain briefly, **ask to sync**, stop
until confirmed or waived.

### 2. Resolve PR with `gh`

```bash
gh pr list --head <branch> --state open --json number,title,url,baseRefName,headRefName
```

| Count | Action |
|-------|--------|
| 0 | No open PR for this head. Stop. |
| 1 | Use that PR. |
| 2+ | List `number`, `title`, `url`. Ask which. Wait. |

Then:

```bash
gh pr view <n> --json title,body,baseRefName,headRefName,commits,comments,reviews,reviewDecision,url,author,files
gh api repos/{owner}/{repo}/pulls/<n>/comments
gh api repos/{owner}/{repo}/issues/<n>/comments
git merge-base HEAD origin/<baseRefName>
git diff --stat <merge-base>...HEAD
git diff <merge-base>...HEAD
```

`{owner}/{repo}` from `gh repo view --json nameWithOwner -q .nameWithOwner`.

### 3. Code review (→ table **C**)

Apply the **`review-branch`** skill rules (read that skill if needed): AGENTS /
README / sibling patterns, regressions (including silent CSS cascade /
specificity / overflow), globals, reuse, naming, magic values, dead code,
diff-scoped a11y/i18n/CI/secrets, and **Manual regression checks** when UI
CSS/markup or interactive overlays moved.

Reuse a `/review-branch` result from **this chat** only if it matches the
current branch tip; otherwise run the review now. Map Critical → ⛔️, Watchouts
→ ⚠️. Cap **8** rows. Evidence required. Carry forward that skill’s
**Manual regression checks** section into this reply (same steps; do not drop
them when compressing into tables).

### 4. PR review (→ table **P**)

Evaluate against the current diff:

1. Description vs diff (missing claims, unstated scope, stale/empty body)
2. Open feedback still applicable (`CHANGES_REQUESTED`, unresolved threads)
3. Conversation constraints the description omits and the diff may ignore

⛔️ = must resolve before merge; ⚠️ = worth noting. Cap **8**. Evidence required.

### 5. Ticket review (→ table **T**)

Detect Linear IDs (`A-Z+`-`digits`) from PR title, body, branch, commits.

| Condition | Action |
|-----------|--------|
| No IDs, or Linear not usable | Table **T** with one None row; one-line skip note. |
| IDs and Linear usable | Fetch ticket(s); review implementation vs description / AC / status. |

Gaps only with ticket + diff evidence. ⛔️ = missing/broken AC; ⚠️ = soft
mismatch. Cap **8**.

## Output format

Always emit **all three** tables, in this order, then **Manual regression
checks** from the code pass when `review-branch` would emit them. Keep
Description and Suggestion to **one short line each**. No essays, no patches.

```text
PR: #<n> <url>
Base: <baseRefName> ← <headRefName>
Sync: ok | waived
```

### Code

| Number | Type | Description | Reference | Suggestion |
|--------|------|-------------|-----------|------------|
| C1 | ⛔️ or ⚠️ | Short issue | `` `path:line` `` · vs origin/main | Short action |

### PR

| Number | Type | Description | Reference | Suggestion |
|--------|------|-------------|-----------|------------|
| P1 | ⛔️ or ⚠️ | Short mismatch / open feedback | [comment](url) or `` `path:line` `` | Short action |

### Ticket

| Number | Type | Description | Reference | Suggestion |
|--------|------|-------------|-----------|------------|
| T1 | ⛔️ or ⚠️ | Short gap vs ticket | [`TEAM-123`](ticket-url) + `` `path:line` `` | Short action |

**Numbering:** `C1…Cn`, `P1…Pn`, `T1…Tn` in priority order within each table.

**Empty table:** one row `| — | — | None | — | — |`.

**Reference — code (required shape):**

- Always a **local workspace path with line** (or range): `` `path/to/file.ext:12` ``
  or `` `path/to/file.ext:12-18` ``. Workspace-relative. Never a GitHub/GitLab
  blob or permalink for code — those leave the editor.
- When the file differs from the PR base, append ` · vs origin/<baseRefName>`
  (the same base as the header). That marks the row as diffable.
- AGENTS/README cites stay as plain text section pointers (e.g. `AGENTS.md § CSS`),
  not remote links.
- Non-code only: PR comment / review URLs and Linear issue URLs stay as markdown
  links. If a row also needs code, add `` `path:line` `` beside them.

**Follow-ups (by row number):**

| User says | Do |
|-----------|----|
| `C2` / `expand P1` | Short detail for that row (still no patch unless asked). |
| `diff C1` / `open C1` | For a code row: show the hunk vs `origin/<baseRefName>` for that path (`git diff origin/<base> -- path`), focused on the cited lines. Prefer the editor-friendly local path in the reply so they can click through. |

**Type:**

| Symbol | Meaning |
|--------|---------|
| ⛔️ | Blocking — fix or explicitly waive before merge |
| ⚠️ | Warning — non-blocking, still worth a look |

After the three tables, if the code pass produced **Manual regression checks**,
emit that section unchanged (or `None (no UI CSS or interactive markup in diff)`).

End with: review file diffs manually; you own commits and messages. Remind that
`diff <id>` opens the change vs the PR base for that row.

## Hard rules

- Do not invent PR numbers, comment text, ticket AC, or evidence.
- Do not open, edit, or merge the PR.
- Do not put GitHub/blob URLs in Reference for code — local `path:line` only.
- Prefer fewer evidenced rows over speculative noise.
- Not a substitute for Bugbot or Security Review.
