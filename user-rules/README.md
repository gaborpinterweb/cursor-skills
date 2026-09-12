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