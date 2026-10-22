# Cursor skills

Personal Cursor Agent skills for technical SEO and AEO audits, follow-up fixes, understanding AI diffs, explaining scoped diffs, branch review, PR review, and PR drafting.

## General skills

| Skill name | Description |
|---|---|
| [create-pr](create-pr/) | Drafts a PR message (ticket, changes, decisions, links), asks for confirmation, then creates the PR with gh. |
| [explain-changes](explain-changes/) | Explains intention and behaviour of diffs for a listed file set vs origin/main or origin/master (not a review). |
| [review-branch](review-branch/) | Reviews the current branch against main/master for regressions, globals, reuse, naming, magic values, and AGENTS/README compliance. |
| [review-pr](review-pr/) | Reviews an open PR for the branch (sync gate, gh resolve, optional review-branch from chat, PR alignment, optional Linear vs diff). |

## SEO/AEO skills

| Skill name | Description |
|---|---|
| [technical-seo-audit](technical-seo-audit/) | Runs a technical SEO audit and writes a dated HTML report plus handoff. |
| [technical-seo-implement](technical-seo-implement/) | Implements High/Medium fixes from an SEO audit `handoff.md`. |
| [technical-seo-pr](technical-seo-pr/) | Drafts and creates a technical SEO PR (score sentence, improvements / new-content / FROM→TO tables, test plan) and includes the audit HTML reports. |
| [technical-aeo-audit](technical-aeo-audit/) | Runs a technical AEO audit (Answerlint-oriented) and writes a dated HTML report plus handoff. |
| [technical-aeo-implement](technical-aeo-implement/) | Implements High/Medium fixes from an AEO audit `handoff.md`. |
