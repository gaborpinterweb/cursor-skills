# User rules

User rules in Cursor Settings (Customize → Rules) are global. They apply across all repositories and sync with your account. They apply to Agent (Chat) only (not to Tab, Inline Edit or Bugbot PR reviews).

Skip visual checks

```
Do not visually test implementations in the browser (screenshots, click-throughs, MCP browser tools) unless I explicitly ask. After code changes, a build/lint/log check is enough. Only open the browser when I say so (e.g. “check in the browser”, “take a screenshot”, “verify the UI”).
```

Encourage manual reviews

```
Always encourage me to review the file diffs manually. Remind me that I take responsibility over the code. Encourage me to commit manually with my own commit message. Do it it concisely and short.
```

Split large tasks

```
Before implementing, briefly judge the task: if it likely needs many files/
steps, could hit several open issues or edge cases, or the diff would be
large, don't start yet — propose a short breakdown into smaller, reviewable
steps and ask which to start with. Otherwise just proceed.
```

Plan with Acceptance and Regression tests

```
When planning (Plan mode, or a plan before a multi-step change), propose two kinds of tests:
- Acceptance: proves the desired outcome is achieved. Always required when behavior or user-visible output changes.
- Regression: proves we did not break existing behavior. Propose only if existing coverage (CI, all-checks, build, linkcheck, html-validate, etc.) does not already cover the new risk; otherwise name what already covers it.

For each proposed test, state the check and where it should live (e.g. node --test script, site all-checks/CI, thin Playwright smoke). Prefer CI/all-checks over git hooks for build-heavy checks.

If the plan outlines neither Acceptance nor Regression tests, say so explicitly in the plan with one short reason (e.g. pure content/docs with no behavioral change; or no new Acceptance surface and Regression already covered by existing all-checks). Do not silently omit the testing section.
```
