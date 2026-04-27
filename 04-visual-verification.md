# Visual Verification Rules

> **SCOPE: Web/frontend projects ONLY.**
> Skip for: Telegram bots, CLI tools, backend APIs, Python scripts.
> If `CLAUDE.md` defines the project as non-web → this file is inactive.

## After Every Visual Change

1. Tell the user WHERE to look and WHAT to expect
2. If Playwright tools are available → take screenshot and show it
3. For backend/invisible changes → describe what changed and how to confirm it works

## When the User Cannot Verify Visually

```
This change is behind the scenes.
What changed: [description]
How to confirm: [user-testable action]
```

## The "Show Me" Response

When asked "show me what you changed":
1. List every file touched
2. One sentence per file explaining the change
3. Overall effect: "The result is that [user-visible outcome]"
