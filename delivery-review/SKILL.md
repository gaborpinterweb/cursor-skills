---
name: delivery-review
description: >-
  Delivery overview of the open PR for the current (or named) branch: code
  review (branch/AGENTS compliance), PR review (description and feedback vs
  diff), and Linear ticket review vs implementation. Emits three concise
  tables (C# / P# / T#). Use when the user runs /delivery-review or /review-pr,
  asks for a delivery review, or wants a pre-merge overview.
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
README / sibling patterns, regressions, globals, reuse, naming, magic values,
dead code, diff-scoped a11y/i18n/CI/secrets.

Reuse a `/review-branch` result from **this chat** only if it matches the
current branch tip; otherwise run the review now. Map Critical → ⛔️, Watchouts
→ ⚠️. Cap **8** rows. Evidence required.

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

Always emit **all three** tables, in this order. Keep Description and Suggestion
to **one short line each**. No essays, no patches.

```text
PR: #<n> <url>
Base: <baseRefName> ← <headRefName>
Sync: ok | waived
```

### Code

| Number | Type | Description | Reference | Suggestion |
|--------|------|-------------|-----------|------------|
| C1 | ⛔️ or ⚠️ | Short issue | [`path:line`](link) or AGENTS cite | Short action |

### PR

| Number | Type | Description | Reference | Suggestion |
|--------|------|-------------|-----------|------------|
| P1 | ⛔️ or ⚠️ | Short mismatch / open feedback | [PR body](url) / comment / `path` | Short action |

### Ticket

| Number | Type | Description | Reference | Suggestion |
|--------|------|-------------|-----------|------------|
| T1 | ⛔️ or ⚠️ | Short gap vs ticket | [`TEAM-123`](ticket-url) + `path` | Short action |

**Numbering:** `C1…Cn`, `P1…Pn`, `T1…Tn` in priority order within each table.

**Empty table:** one row `| — | — | None | — | — |`.

**Reference:** must be clickable when a URL or path link is available (repo file
blob/permalink, PR comment URL, Linear issue URL). Plain `path:line` only if no
link can be built.

**Type:**

| Symbol | Meaning |
|--------|---------|
| ⛔️ | Blocking — fix or explicitly waive before merge |
| ⚠️ | Warning — non-blocking, still worth a look |

End with: review file diffs manually; you own commits and messages. Offer to
expand any row by number on request.

## Hard rules

- Do not invent PR numbers, comment text, ticket AC, or evidence.
- Do not open, edit, or merge the PR.
- Prefer fewer evidenced rows over speculative noise.
- Not a substitute for Bugbot or Security Review.
