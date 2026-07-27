# technical-aeo-implement

Cursor skill that applies fixes from a **technical-aeo-audit** handoff. It implements code changes; it does **not** re-run the full ~47-rule audit from scratch.

## When to use it

After an AEO audit, ask Cursor to implement the AEO handoff / apply AI-visibility recommendations / fix High and Medium IMP tasks.

## How it works

1. Reads `aeo-audit/YYYY-MM-DD-HHmm/handoff.md` (ask for the path if missing).
2. Uses `index.html` / `meta.json` / attachments only for context when helpful.
3. Implements **High**, then **Medium** `IMP-*` tasks. Skips **Low** unless you ask for full polish.
4. For each task: meets **Acceptance**, follows existing project patterns, keeps the diff focused.
5. Preserves intentional audit notes (bot policy, `n/a` FAQ/comparison pages, etc.).
6. Finishes with a short summary (optionally `implementation-summary.md` in the audit folder) and suggests a fresh **technical-aeo-audit** re-run.

## Required input

```text
aeo-audit/YYYY-MM-DD-HHmm/handoff.md
```

Also useful: that folder’s `index.html`, `meta.json`, and Answerlint attachments.

## Guardrails

- Do not invent scope beyond `handoff.md` unless you expand it in the prompt.
- Do not invent FAQ/HowTo schema that is not visible on the page.
- Prefer shared layouts/partials over one-off page edits.
- No drive-by refactors.

## Key files in this skill

| File | Role |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent instructions (source of truth for Cursor) |

## Pairing

Produced by [`technical-aeo-audit`](../technical-aeo-audit/). After implementing, re-audit in a new dated folder (ideally with fresh Answerlint JSON/CSV).
