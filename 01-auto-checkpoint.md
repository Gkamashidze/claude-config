# Auto-Checkpoint & Backup System

> **Project CLAUDE.md can override checkpoint frequency and file-count limits.**
> If CLAUDE.md grants autonomous execution → skip confirmation steps.
> Hard limits (never delete, never force-push) remain regardless.

## When Claude MUST Auto-Commit (No Permission Needed)

1. **Before any multi-file change** (3+ files)
2. **Before touching configuration files** (pyproject.toml, railway.toml, .env, docker-compose, etc.)
3. **After completing a working feature**
4. **Before any refactoring operation**
5. **Before installing or removing dependencies**
6. **Before modifying database schemas or migrations**
7. **At the end of every session** if there are uncommitted changes

## When Claude SHOULD Auto-Commit (Recommended)

- After every 2 successful file edits
- After a verified bug fix
- Before switching tasks

## Commit Message Format

```
CHECKPOINT: [description]

What was done: [1-2 sentences]
Files changed: [filenames only]
Status: [WORKING | IN-PROGRESS | EXPERIMENTAL]

Co-Authored-By: Claude Code <noreply@anthropic.com>
```

## File Count Thresholds

If project CLAUDE.md grants autonomous execution → **skip thresholds below**.

Otherwise:
- **3 files** = checkpoint REQUIRED before continuing
- **5 files** = checkpoint REQUIRED + summarize to user
- **10+ files** = checkpoint + summarize + ask permission

## Auto-Commit Command

```bash
git add <file1> <file2> <file3> && git commit -m "$(cat <<'EOF'
CHECKPOINT: [description]

What was done: [plain language]
Files changed: [list]
Status: [WORKING|IN-PROGRESS|EXPERIMENTAL]

Co-Authored-By: Claude Code <noreply@anthropic.com>
EOF
)"
```

Never use `git add -A` or `git add .` — name files explicitly to avoid staging `.env` or credentials.

## Restore from Checkpoint

When user says "go back", "undo", "restore", "revert":

1. `git log --oneline -20` — find recent checkpoints
2. Show options in plain language with timestamps
3. `git checkout <hash> -- .` to restore
4. Create new checkpoint: `CHECKPOINT: Restored to "[description]"`

## File-Level Backups (for critical single files)

For `.env`, database configs, or any file without git coverage:
```bash
cp important-file.txt important-file.txt.backup-$(date +%Y%m%d-%H%M%S)
```

For data files (CSV, JSON) being transformed:
1. Copy original to `.original` version first
2. Work on the copy
3. Delete `.original` only after user confirms result is correct

Backup files older than 7 days → suggest cleanup, never auto-delete.

## When the User Says "I Lost Something"

1. Check `git log` for recent checkpoints
2. Check for `.backup`, `.original`, `.old` files
3. If not found: "I cannot find a backup of that file. It may not have been saved."
