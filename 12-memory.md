# Memory & Context Management Rules

Claude MUST follow these rules to maintain perfect context across sessions.

---

## 1. SESSION START PROTOCOL

At the beginning of EVERY session:

1. CLAUDE.md loads automatically — read it silently, do not recite it
2. If auto-memory files exist → read them for prior context
3. If handoff note exists (`.claude/handoff-*.md`) → read it and resume
4. Dive straight into the user's request — do not ask "what would you like to work on?"

---

## 2. SESSION END PROTOCOL

Before the session ends (user says goodbye, or context is getting full):

1. Create a handoff note: `.claude/handoff-YYYY-MM-DD.md`
   - What was accomplished this session
   - Current state of the project
   - What's left to do (next steps)
   - Any important discoveries or decisions made
   - Unresolved issues or bugs
2. If user preferences were learned → save to auto-memory
3. Suggest: "Want me to save a checkpoint before we stop?"

---

## 3. WHAT TO REMEMBER (and WHERE)

| Information | Where to Store |
|---|---|
| Build/test commands that work | CLAUDE.md |
| User preferences & feedback | Auto memory (`~/.claude/projects/.../memory/`) |
| What was built and why | git commit messages |
| Session context for next time | `.claude/handoff-*.md` |
| Architecture decisions | CLAUDE.md (brief) or commit messages |

---

## 4. CONTEXT WINDOW MANAGEMENT

- Compact BEFORE reaching 60% context usage, not after
- If conversation is getting long (20+ exchanges): suggest `/compact`
- If switching to a completely different topic: suggest `/clear`
- When compacting, preserve: current task state, recent decisions, active bugs
- When compacting, discard: failed attempts that were reverted, verbose tool output already processed

Tell the user: "Our conversation is getting long. I'll compress the earlier parts so we don't lose context."

---

## 5. HANDOFF NOTE FORMAT

```markdown
# Session Handoff - YYYY-MM-DD

## Accomplished
- [List of things completed this session]

## Current State
- App status: [working / has issues / in progress]
- Last commit: [summary]

## Next Steps
1. [Most important next task]
2. [Second priority]

## Important Context
- [Non-obvious information the next session needs]
- [Known issues]
```

---

## 6. WHAT CLAUDE SHOULD NEVER FORGET

Even across sessions, always remember:
- The project's purpose (from CLAUDE.md overview)
- The tech stack (from CLAUDE.md)
- Security rules (never put secrets in code)
- Project CLAUDE.md overrides global rules (see `00-project-overrides.md`)

---

## 7. CLEANING OLD CONTEXT

- Keep the latest 3 handoff notes, suggest cleanup of older ones (never auto-delete)
- Auto memory: Claude manages automatically; trim if MEMORY.md index exceeds 200 lines
