# Development Workflow Standards

## 1. Research Before Writing Code

Before implementing any non-trivial feature:

1. **Search the existing codebase first** — find similar patterns already implemented
2. **Check library docs** — verify API behavior and current versions (PyPI, npm docs, GitHub)
3. **Port proven patterns** over building from scratch

## 2. Planning Phase

For any change touching 3+ files:

1. Define scope: what changes, what stays
2. Identify dependencies and risks
3. Break into phases with checkpoints
4. Consider: data model → business logic → API/handler → tests

## 3. TDD Workflow (RED → GREEN → IMPROVE)

For features and bugfixes:

1. **RED**: Write a failing test describing the desired behavior
2. **GREEN**: Write minimum code to make it pass
3. **IMPROVE**: Refactor while keeping tests green

For async code (aiogram handlers, asyncpg queries):
- Use `pytest-asyncio` with `@pytest.mark.asyncio`
- Mock external services (Telegram API, Anthropic API) — never hit real APIs in tests
- Test the handler logic, not Telegram's delivery

For bug fixes:
1. Write a test reproducing the bug first
2. Fix the bug
3. Confirm test passes — regression prevention forever

## 4. Self-Review Before Commit

1. Re-read every changed file
2. Run full test suite + linter + type checker
3. Check: security, edge cases, error paths
4. Fix ALL issues before committing

## 5. Commit Standards

Conventional commits:
```
type: description
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

Examples:
- `feat: add WAC display in OEM found message`
- `fix: handle TelegramRetryAfter in bulk sender`
- `refactor: extract parser logic to separate module`

## 6. Branch Strategy

> **Project CLAUDE.md can override this.**
> If CLAUDE.md grants "commit and push to main directly" → this section does not apply.

- `main` — production-ready only
- `feature/description` — new features
- `fix/description` — bug fixes

## 7. Pre-Commit Checklist

```
□ All tests pass
□ Linter clean (ruff / eslint)
□ No type errors (mypy / tsc)
□ No debug print/log statements
□ No hardcoded secrets or URLs
□ Commit message follows convention
```

## 8. Model Selection

- **Haiku**: Frequent/cheap invocations, classification, routing
- **Sonnet**: Coding, structured output, multi-step tasks
- **Opus**: Deep reasoning, architecture planning, ambiguous requirements

## 9. Context Management

- Use `/compact` before reaching 60% context usage
- For large changes: break into smaller sessions with handoff notes
