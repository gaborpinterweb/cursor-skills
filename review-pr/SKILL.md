---
name: review-pr
description: >-
  Reviews an open PR for the current (or named) branch: sync gate, resolve via
  gh, optionally reuse /review-branch from chat, then align PR description and
  open feedback against the diff, and optionally review Linear tickets vs
  implementation. Use when the user runs /review-pr, asks to review a PR, or
  wants a pre-merge PR context check.
disable-model-invocation: true
---

# Review PR

Review-only PR context check. Do **not** fix code, edit the PR, or run
`/review-branch` unless the user explicitly asks. Prefer `gh` over inventing MCP
calls for GitHub.

## Workflow

### 1. Sync gate

Resolve branch: user-named branch, else current branch (`git branch --show-current`).

Compare local tip to remote:

```bash
git fetch origin <branch> 2>/dev/null || true
git rev-parse HEAD
git rev-parse @{u} 2>/dev/null || git rev-parse origin/<branch>
git status -sb
```

If local HEAD and remote tip **differ** (ahead, behind, or diverged): explain
briefly and **ask the user to sync**. Stop until they confirm sync is done, or
explicitly waive. Do not continue the review while tips disagree unless waived.

### 2. Resolve PR with `gh`

```bash
gh pr list --head <branch> --state open --json number,title,url,baseRefName,headRefName
```

| Count | Action |
|-------|--------|
| 0 | Explain: no open PR for this head. Stop. |
| 1 | Use that PR. |
| 2+ | List `number`, `title`, `url`. Ask which PR. Wait. |

After settle, load context:

```bash
gh pr view <n> --json title,body,baseRefName,headRefName,commits,comments,reviews,reviewDecision,url,author,files
gh api repos/{owner}/{repo}/pulls/<n>/comments
gh api repos/{owner}/{repo}/issues/<n>/comments
```

Resolve `{owner}/{repo}` via `gh repo view --json nameWithOwner -q .nameWithOwner`.
Also gather the merge-base diff vs the PR base (for alignment):

```bash
git merge-base HEAD origin/<baseRefName>
git diff --stat <merge-base>...HEAD
git diff <merge-base>...HEAD
```

### 3. Optional `/review-branch` (never auto-run)

Scan **this conversation** for an existing `/review-branch` (or equivalent
pattern/AGENTS compliance) result for this branch or PR.

- **Found**: reuse it as the branch-compliance section. Note it came from chat.
  Do not re-run unless the user asks.
- **Not found**: ask whether they want to run `/review-branch` first. Wait.
  - Yes → run the `review-branch` skill (or tell them to invoke `/review-branch`),
    then continue with its output in the reply.
  - No → omit that section; continue.

Do **not** duplicate `review-branch` rules or invent a parallel compliance review
inside this skill.

### 4. PR context review (alignment)

Evaluate:

1. **Description vs diff** — claimed changes missing from the diff; diff scope
   not reflected in the body; empty/stale summary; wrong base assumptions.
2. **Open feedback** — unresolved review threads, `CHANGES_REQUESTED`, or open
   inline comments still applicable to current code (or clearly outdated).
3. **PR conversation** — issue comments and reviews that add constraints or
   decisions the description omits and the diff may ignore.

Review-only. Cap **8** findings. Every row needs evidence. If none:

`| — | — | None | — | — |`

#### Output (this section)

```text
PR: #<n> <url>
Base: <baseRefName> ← <headRefName>
```

### PR context

| ID | Location | Issue | Evidence | Recommended fix |
|----|----------|-------|----------|-----------------|
| 3.1 | `PR body` / `path:line` / comment URL | Concrete mismatch or open feedback | Quote or path | One-line action |

Number **3.1**, **3.2**, … (reserve 1.x/2.x for a reused `/review-branch` block).

### 5. Linear gate (ask before fetch)

Detect Linear-shaped IDs (`A-Z+`-`digits`, e.g. `ENG-123`, `TEAM-42`) from PR
title, body, branch name, and commits (same shape as `create-pr`).

Check Linear usability:

1. A Linear MCP namespace is present and authenticated, **or**
2. GitKraken / issue tools can load Linear issues (`issues_get_detail` or equivalent)

| Condition | Action |
|-----------|--------|
| No IDs, or Linear not usable | Skip Linear section (optional one-line note). |
| IDs **and** Linear usable | List IDs. **Ask** before fetching. Wait. |

On yes: fetch ticket(s), then review **implementation vs ticket** (description,
acceptance criteria, status). Flag gaps only with evidence from ticket + diff.
Cap **8** rows. On no: skip.

#### Output (only if fetched)

### Linear

| ID | Ticket | Gap | Evidence | Recommended fix |
|----|--------|-----|----------|-----------------|
| 4.1 | `TEAM-123` | What is missing or mismatched | Ticket field + `path:line` | One-line action |

### 6. Final reply shape

Include sections only when they exist, in order:

1. Sync note (ok / waived) + `PR: #<n> …` + base/head
2. Reused or just-run `/review-branch` block — or one line: `review-branch: skipped`
3. **PR context** table
4. **Linear** table (only if fetched)
5. Short reminder: review file diffs manually; you own commits and messages

## Hard rules

- Do not invent PR numbers, comment text, or ticket AC.
- Do not open, edit, or merge the PR.
- Do not auto-run `/review-branch`.
- Prefer fewer evidenced rows over speculative noise.
- Not a substitute for Bugbot or Security Review.
