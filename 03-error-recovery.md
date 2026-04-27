# Error Recovery Rules

## When Something Breaks

If an error occurs after Claude makes a change, follow this procedure:

### Step 1: Assess Severity
| Severity | Symptoms | Action |
|----------|----------|--------|
| **Minor** | One feature not working, styling off | Fix silently, inform user what happened |
| **Medium** | Page not loading, form broken | Stop, explain in plain language, fix with permission |
| **Critical** | App won't start, blank screen, data loss risk | IMMEDIATELY restore last checkpoint, then explain |

### Step 2: Communicate Clearly

Use this template when reporting errors:

```
Error: [one sentence — what broke and why]
Impact: [what the user sees / what doesn't work]
Action: [what I'm doing to fix it]
Data: [SAFE / AT RISK — if at risk, explain]
```

Be direct and technical — use real error names, file paths, and status codes. Example:
- "TelegramRetryAfter raised in bulk sender — sleeping 30s then retrying."
- "asyncpg.UniqueViolationError on product insert — barcode already exists in DB."
- "mypy type error in handler — fixing and re-running."

### Step 3: Fix Procedure

1. **If it is a simple typo or small mistake**: Fix it immediately, verify it works, tell the user
2. **If the fix is unclear**: Restore the last working checkpoint FIRST, then investigate
3. **If multiple things broke**: Do NOT try to fix them all at once. Restore checkpoint. Start over with a smaller change.
4. **If you cannot fix it after 2 attempts**: Stop. Explain what's wrong, what you tried, and what the next step is. Suggest restoring the last checkpoint.

### Step 4: Post-Recovery

After any error recovery:
1. Create a new checkpoint of the restored/fixed state
2. Summarize what went wrong in one sentence
3. Explain what you will do differently this time
4. Proceed more cautiously (smaller changes, more verification)

## Auto-Diagnosis Triggers

When the user says any of these, enter diagnosis mode:
- "Something broke"
- "It's not working"
- "The page is blank/white"
- "I see an error"
- "It looks wrong"
- "That's not what I wanted"

### Diagnosis Procedure
1. Check the most recent changes Claude made
2. Run the app/site and check for errors
3. Compare current state to the last working checkpoint
4. Present findings in plain language
5. Offer: "Should I fix it or go back to the last save?"

## The Two-Strike Rule

If Claude's fix attempt fails twice:
- STOP trying the same approach
- Restore to the last working checkpoint
- Restore to last working checkpoint and explain what failed and why a different approach is needed.

## Preventing Cascading Failures

- NEVER make a second change to fix a broken first change without checkpointing
- NEVER modify additional files to "work around" an error
- NEVER suppress, hide, or comment out errors -- fix the root cause
- If fixing one thing breaks another, restore and rethink the entire approach
