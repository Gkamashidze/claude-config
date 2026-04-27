# Automated Testing System for Non-Technical Users

> Claude MUST follow these rules automatically. The user never runs tests manually.
> All testing is invisible to the user — they only see results, screenshots, and plain-language summaries.
>
> **UI/Visual testing (Sections 2, 5-step 3, 9, 11):** Web/frontend projects ONLY.
> Skip Playwright screenshots for Telegram bots, CLIs, backend APIs, Python scripts.
> Code testing (unit, integration) applies to ALL project types.

---

## 1. WHEN TO TEST (Automatic Triggers)

These triggers are **mandatory** — Claude executes them without being asked:

| Trigger | Action |
|---------|--------|
| Created/modified any `.tsx`, `.vue`, `.html`, `.css`, `.scss` file | Visual screenshot verification (see Section 2) |
| Created/modified any API route, endpoint, or server function | Run endpoint test (see Section 4) |
| Any bug fix requested | Write regression test FIRST → then fix → verify test passes |
| Before every `git commit` | Run full test suite; block commit if tests fail |
| After `npm install`, `pip install`, or dependency update | Run existing tests to verify nothing broke |
| Created any new function, utility, or module | Write at least 1 unit test (happy path + 1 error case) |
| Modified authentication or authorization logic | Run full auth flow test |
| Modified database schema or queries | Run CRUD operation tests |

### Auto-Detection Rules
- If the project has `vitest.config.*` or `jest.config.*` → use that framework
- If the project has `pytest.ini`, `pyproject.toml` with pytest, or `conftest.py` → use pytest
- If the project has `playwright.config.*` → use Playwright E2E
- If no test framework exists and project has 3+ source files → set up Vitest (Node) or Pytest (Python) automatically
- Always check for existing test patterns in the project and follow them

---

## 2. UI/UX TESTING (Playwright MCP)

After any visual change: take desktop (1440×900), mobile (375×812), tablet (768×1024) screenshots.
Check for broken layouts, overflow, unreadable text, console errors, failed network requests.
Show screenshots to user. Ask: "ნახე შედეგი — კარგად გამოიყურება?"
Full Playwright protocol: see `11-ui-verification.md`.

---

## 3. CODE TESTING

### Unit Test Requirements:
- **Every new function** gets at least 1 test (happy path + 1 error case)
- **Every bug fix** gets a regression test written BEFORE the fix
- **Every utility/helper** gets edge case tests (empty input, null, undefined, boundary values)

### Test Coverage Minimums:
- New files: 80%+ line coverage
- Critical paths (auth, payments, data mutations): 90%+ coverage
- Utility functions: 100% coverage

### What to Test:
```
Happy path:        Function returns expected output for valid input
Error handling:    Function handles invalid input gracefully (null, undefined, empty string)
Edge cases:        Boundary values, large inputs, special characters
Input validation:  Empty strings, XSS attempts (`<script>alert('xss')</script>`), SQL injection patterns
Type safety:       Wrong types passed to functions (number where string expected)
```

### What to Mock:
- External API calls (Supabase, Stripe, third-party APIs)
- Database connections
- File system operations
- Environment variables
- Time-dependent functions (use fake timers)

### Test File Naming:
- Place tests next to source: `component.tsx` → `component.test.tsx`
- Or in `__tests__/` directory following project convention
- Match existing project patterns — do NOT introduce a new convention

---

## 4. INTEGRATION TESTING

### API Endpoint Tests:
For every API route, verify:
- Returns correct status codes (200, 201, 400, 401, 403, 404, 500)
- Returns correct response shape/schema
- Handles missing/invalid parameters with proper error messages
- Authentication/authorization is enforced
- Rate limiting works (if applicable)

### Authentication Flow Tests:
```
1. Login with valid credentials → success
2. Login with invalid credentials → proper error
3. Access protected route without token → 401
4. Access protected route with expired token → 401
5. Access admin route as regular user → 403
6. Logout → token invalidated
```

### Database Operation Tests:
```
1. Create record → record exists in DB
2. Read record → returns correct data
3. Update record → changes persisted
4. Delete record → record removed
5. Duplicate/conflict handling → proper error
```

### Webhook/n8n Integration Tests:
- Send test payload to webhook endpoint → verify 200 response
- Verify webhook data format matches expected schema (`$json.body`)
- Test with malformed payload → proper error handling
- Verify idempotency where applicable

---

## 5. AUTOMATED VERIFICATION FLOW

After Claude makes ANY code change, execute this sequence automatically:

```
┌─────────────────────────────────────────────┐
│ 1. AUTO-CHECKPOINT                          │
│    Save current state (git stash or commit)  │
├─────────────────────────────────────────────┤
│ 2. RUN UNIT TESTS                           │
│    If fail → fix immediately → re-run       │
│    Do NOT proceed until tests pass           │
├─────────────────────────────────────────────┤
│ 3. UI CHANGED?                              │
│    Yes → Playwright screenshot → show user   │
│    No → skip to step 4                       │
├─────────────────────────────────────────────┤
│ 4. API CHANGED?                             │
│    Yes → test endpoint → show response       │
│    No → skip to step 5                       │
├─────────────────────────────────────────────┤
│ 5. SHOW RESULTS TO USER                     │
│    Screenshots, plain-language summary       │
│    "ნახე შედეგი — კარგად გამოიყურება?"       │
├─────────────────────────────────────────────┤
│ 6. USER APPROVES?                           │
│    Yes → commit with descriptive message     │
│    No → fix and repeat from step 2           │
└─────────────────────────────────────────────┘
```

### If Tests Fail During This Flow:
1. Read the error message
2. Identify the root cause
3. Fix the code (not the test, unless the test is wrong)
4. Re-run tests
5. If still failing after 3 attempts → tell the user what's wrong in plain language
6. Never silently skip failing tests

---

## 6. PRE-COMMIT TESTING HOOK

### Before Every Commit:
```
1. Run linter (eslint/pylint) → fix any issues automatically
2. Run formatter (prettier/black) → apply formatting
3. Run unit tests → ALL must pass
4. Run type checker (tsc/mypy) if configured → no errors
5. Check for console.log/print statements → remove or warn
6. Check for hardcoded secrets/API keys → block if found
7. If ALL pass → proceed with commit
8. If ANY fail → fix → re-run → only commit when clean
```

### Rules:
- NEVER use `--no-verify` to skip hooks
- NEVER commit with failing tests
- NEVER disable or remove test files to make tests "pass"
- If a test is genuinely wrong, fix the test AND document why
- If fixing takes too long (3+ attempts), inform the user before proceeding

---

## 8. TEST MAINTENANCE

### When Modifying Existing Code:
1. Run existing tests FIRST to establish baseline
2. Make the code change
3. Run tests again — if tests fail, determine:
   - Is the test outdated? → Update the test
   - Is the code wrong? → Fix the code
4. Add new tests for new behavior

### Test Cleanup:
- Remove tests for deleted features
- Update test descriptions when behavior changes
- Keep test files organized (match source structure)
- Never leave commented-out tests

---

## 10. QUICK REFERENCE: Test Commands by Project Type

| Project Type | Unit Tests | E2E Tests | Lint |
|-------------|-----------|-----------|------|
| Next.js/React | `npx vitest run` or `npx jest` | `npx playwright test` | `npx eslint .` |
| Vue/Nuxt | `npx vitest run` | `npx playwright test` | `npx eslint .` |
| Python/FastAPI | `python -m pytest` | `python -m pytest tests/e2e/` | `pylint src/` |
| Python/Django | `python manage.py test` | `python -m pytest tests/e2e/` | `pylint .` |
| Static HTML/CSS | N/A | Playwright MCP visual check | N/A |

> These commands are for Claude's internal use. Never show them to the user.
