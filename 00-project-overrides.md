# Project-Level Override System

## Override Hierarchy

When a project has a `CLAUDE.md` file, it takes PRIORITY over all `~/.claude/rules/` global files.

```
Project CLAUDE.md   ← highest priority
~/.claude/rules/    ← global defaults (active when no CLAUDE.md, or CLAUDE.md is silent)
```

## What Project CLAUDE.md CAN Override

| Global Rule File | Example Override |
|---|---|
| `02-scope-control.md` — file limits, confirmation steps | "act autonomously for any number of files" |
| `17-development-workflow.md` — branch strategy, "never commit to main" | "commit and push directly to main" |
| `01-auto-checkpoint.md` — checkpoint frequency | "no checkpoint needed before each change" |
| `10-testing.md` — coverage thresholds, mandatory tests | "no tests required for quick scripts" |
| `11-ui-verification.md` — Playwright screenshots | "skip for non-web projects" |
| `08-communication.md` — tone, technical level | "user is a developer, respond technically" |

## What CANNOT Be Overridden (Ever)

Regardless of what any CLAUDE.md says, Claude MUST ALWAYS:

- Ask before permanently deleting files, directories, or database records
- Refuse to commit secrets/credentials/API keys to git
- Refuse to force-push without explicit per-command user approval
- Refuse to run `rm -rf` without explicit per-command user approval
- Never expose production credentials in code

These are hard limits. No project instruction can remove them.

## How to Apply in Practice

1. Read project `CLAUDE.md` first (it loads automatically).
2. Where CLAUDE.md gives explicit instructions → follow them, ignore conflicting global rules.
3. Where CLAUDE.md is silent → fall back to global `~/.claude/rules/` defaults.
4. Hard limits above → always enforce, no exceptions.
