---
description: Revert a track, phase, or task using git-aware rollback
argument-hint: [track|phase|task] [name]
allowed-tools: Read, Bash, Glob, Grep
model: claude-haiku-4-5-20251001
---

# Conductor Revert

Git-aware revert that understands logical units of work (tracks, phases, tasks).

## Pre-flight Checks

1. Verify `conductor/tracks.md` exists. If not: "Run `/conductor-okf:setup` first."
2. Check `git status --porcelain`. If uncommitted changes: "Commit or stash before reverting."

## Phase 1: Target Selection

### Parse Arguments

| Pattern | Action |
|---------|--------|
| `track <name>` | Revert entire track |
| `phase <name>` | Revert specific phase |
| `task <name>` | Revert specific task |
| (no args) | Show menu of in-progress/recent items |

### If no target provided

1. Scan `tracks.md` and `plan.md` files for `[~]` (in-progress) and recent `[x]` (completed) items
2. Use AskUserQuestion with numbered options for candidates + "A different item" option
3. If "different", ask for specific name

### Confirm Target

Use AskUserQuestion: "Revert [Phase] 'Backend API' from track 'user_auth'?" Yes/No

## Phase 2: Git Analysis

**Run these git operations in parallel (multiple Bash calls in one response):**

1. **Extract commit SHAs** from target's plan.md:
   ```bash
   grep -o '\[[x~]\].*\[[a-f0-9]\{7\}\]' conductor/tracks/<id>/plan.md
   ```

2. **Find plan update commits:**
   ```bash
   git log --oneline -- conductor/tracks/<id>/plan.md
   ```

3. **For track reverts - find creation commit:**
   ```bash
   git log --oneline -- conductor/tracks.md | grep -i "add track"
   ```

**Sequential verification:**
- After parallel reads, verify each extracted SHA exists: `git cat-file -t <sha>`
- If missing, search by message: `git log --oneline --all | grep "<message>"`
- Compile ordered list (newest first) of all commits to revert (implementation + plan updates)

> **Performance note:** Parallelizing git read operations speeds up revert preparation.

## Phase 3: Execution Plan

### Present Summary

Display:
- Target description
- List of commits (newest first) with messages
- Action: `git revert --no-edit` on each
- Warnings: Merge commits need special handling, old commits may have dependents

Use AskUserQuestion: "Proceed with revert?" Yes/No

## Phase 4: Execution

### Execute Reverts

For each commit (newest to oldest):
```bash
git revert --no-edit <sha>
```

### Handle Conflicts

If conflict: Use AskUserQuestion: "Show conflict for manual resolution / Abort revert"
- If show: Display conflict, wait for resolution, run `git add . && git revert --continue`
- If abort: Run `git revert --abort`, announce rollback

### Verify Plan State

After reverts:
1. Read affected plan.md
2. If status doesn't match (still `[x]` but code reverted):
   - Edit to change `[x]` → `[ ]`
   - Remove commit SHAs
   - Commit: `git commit -m "conductor(plan): Reset status after revert"`

## Completion

**Success:**
Announce:
- Reverted N commits
- Reset target status
- Changed files list
- Next steps: Run tests, `/conductor-okf:status`, `/conductor-okf:implement` to restart

**Partial:**
Announce:
- Successfully reverted commits
- Failed commits (conflicts)
- Manual intervention needed
- Run `git status` for details
