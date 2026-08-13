---
name: create-pr
description: >-
  Drafts a pull request title and body from branch commits (ticket IDs, changes,
  decisions, preview/ticket links), asks for confirmation, then creates the PR
  with gh. Use when the user asks to create a PR, open a pull request, or run
  /create-pr.
disable-model-invocation: true
---

# Create PR

Draft first, create only after explicit confirmation. Never open the PR until the
user approves the message (and any missing links they choose to add).

## Workflow

### 1. Gather context

Run in parallel:

- `git status`
- `git diff` (staged + unstaged) and `git diff <base>...HEAD` when branched
- `git log <base>..HEAD` (full messages) and `git log <base>..HEAD --oneline`
- Branch tracking / ahead-behind vs remote
- Prefer the same base-branch rules as `review-branch`: fresher of `main` /
  `master`, unless the user names a base

Also check branch name and commit subjects/bodies for ticket IDs.

### 2. Extract ticket IDs

Scan commits, branch name, and (if present) recent chat for tracker keys:

| Tracker | Patterns (examples) |
|---------|---------------------|
| Jira | `PROJ-123`, `ABC-456` |
| Linear | `ENG-123`, `TEAM-42` (same `A-Z+`-`digits` shape) |
| GitHub Issues | `#123`, `GH-123`, `owner/repo#123` |
| Other | `fixdesk-123`, `asana-…` only if clearly ticket-like in commits |

Deduplicate. Prefer the most specific ID that appears in commits. If several
unrelated IDs appear, list them all and ask which belongs on the PR.

Build ticket URL when possible:

- Jira: from repo docs/`AGENTS.md`/env hints, or existing remote issue links in
  commits; else leave URL empty and ask
- Linear: `https://linear.app/<workspace>/issue/<ID>` only if workspace is known
  from repo config or prior context; else ask
- GitHub: `https://github.com/<owner>/<repo>/issues/<n>`

### 3. Preview link

Try, in order:

1. User-provided preview URL in the request
2. Bot/CI comments on the branch’s existing PR (if any) — Vercel/Netlify/Cloudflare
3. Repo docs for preview URL patterns

If none: do **not** invent a URL. In the confirmation step, suggest adding a
preview link (and ticket link if missing).

### 4. Draft the PR message

**Title:** short, imperative; include ticket ID when found
(`ABC-123: add footer CTA`). Match repo commit/PR style from `git log`.

**Body** — use this shape (omit empty optional link lines only after the user
confirms they should be omitted):

```markdown
## Summary
<1–3 sentences: why this PR exists>

## Changes
- <max 8 bullets: user-visible or structural changes>

## Decisions
- <max 8 bullets: non-obvious choices a reviewer should know>

## Links
- Ticket: <url or ID>
- Preview: <url>
```

Rules:

- **Changes**: max **8**. Concrete outcomes, not file lists. Merge related edits.
- **Decisions**: max **8**. Tradeoffs, intentional globals, “why not X”, scope
  cuts. Skip if nothing noteworthy — write `None.` as a single bullet.
- Do not dump the full diff. Do not invent decisions that are not evidenced by
  commits/diff/chat.
- Include a brief **Test plan** checklist only if the repo’s recent PRs usually
  have one; otherwise skip unless the user asks.

### 5. Ask for confirmation (required)

Show the user:

1. Base branch and head branch
2. Proposed **title**
3. Proposed **body** (full markdown)
4. Detected ticket IDs and resolved/unresolved links
5. Explicit prompts when links are missing, e.g.:
   - “No preview URL found — paste one to include, or confirm create without it.”
   - “Ticket `ABC-123` found but no URL — paste the Linear/Jira link, or confirm ID-only.”

Wait for confirmation. Accept edits to title/body/links. Do **not** run
`gh pr create` until the user clearly confirms (e.g. “looks good”, “create it”,
“yes”).

### 6. Create the PR (after confirmation only)

1. Push the branch if needed: `git push -u origin HEAD`
2. Create with `gh pr create` using a HEREDOC for the body
3. Return the PR URL

```bash
gh pr create --base <base> --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

If a PR for this branch already exists, update the user with its URL and offer
to edit title/body with `gh pr edit` instead of opening a duplicate.

## Response shape (before create)

Keep the confirmation reply compact:

```markdown
**Branch:** `<head>` → `<base>`

**Title:** …

**Body:**
---
…full draft…
---

**Links:** Ticket: <url|missing> · Preview: <url|missing>

Confirm to create, or send edits / missing links.
```

## Do not

- Create or push before confirmation
- Force-push unless the user explicitly asks
- Invent ticket IDs, preview URLs, or decisions
- Exceed 8 bullets in Changes or Decisions
