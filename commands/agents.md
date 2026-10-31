---
description: Manage background Conductor agents and worktrees
argument-hint: [status|results|worktrees|cleanup]
allowed-tools: Read, Bash, Glob
model: claude-haiku-4-5-20251001
---

# Conductor Agents

Monitor and manage background Conductor agents and git worktrees.

## Commands

### Default (no arguments): Check Background Agent Status

**Run in parallel (single response with multiple Bash calls):**

1. **List running tasks**:
   ```bash
   # Use /tasks command to see all background agents
   # Parse output to identify conductor agents (implementer, planner, reviewer)
   ```

2. **List worktrees**:
   ```bash
   git worktree list
   ```

**Status Report Format:**

For each background agent, display:
- Agent type (implementer/planner/reviewer)
- Task ID or description
- Status (running/completed/failed)
- Worktree path (if using worktree isolation)

**If no background agents:** "No background agents currently running."

**If worktrees exist but no agents:** Suggest `/conductor-okf:agents cleanup` to remove orphaned worktrees.

---

### `/conductor-okf:agents results` - Retrieve Results

Use TaskOutput to retrieve results from completed background agents.

**For each completed agent:**

1. **Retrieve output**:
   ```
   TaskOutput tool:
   - task_id: <agent_task_id>
   - block: false
   ```

2. **Display summary**:
   - **Implementer**: Tasks completed, commits made, checkpoint SHA
   - **Planner**: Track ID, spec/plan paths created
   - **Reviewer**: Verification status, checkpoint SHA

3. **Worktree merge** (if applicable):
   - Check if agent used worktree isolation
   - Merge worktree branch back to main
   - Cleanup worktree (see cleanup section)

---

### `/conductor-okf:agents worktrees` - List Worktrees

Display detailed worktree information:

```bash
git worktree list
```

For each worktree:
- Path: `.worktrees/<agent>-<id>`
- Branch: `conductor/<agent>-<id>`
- Status: Check if branch has uncommitted changes

**Commands for manual management:**

```bash
# View worktree branch commits
git log conductor/<agent>-<id>

# Manually merge worktree
git merge --no-ff conductor/<agent>-<id> -m "conductor(merge): Manual merge of <agent> worktree"

# Remove worktree
git worktree remove .worktrees/<agent>-<id>
git branch -d conductor/<agent>-<id>
```

---

### `/conductor-okf:agents cleanup` - Clean Orphaned Worktrees

Remove worktrees that no longer have running agents.

**Safety protocol:**

1. **List all worktrees**:
   ```bash
   git worktree list
   ```

2. **Check for running agents**:
   - Compare worktrees against active background tasks
   - Identify orphaned worktrees (no associated running agent)

3. **For each orphaned worktree**, ask user:
   - "Worktree `.worktrees/<agent>-<id>` has no running agent. Merge and cleanup? (yes/no)"

4. **If yes**:
   ```bash
   # Check for uncommitted changes
   cd .worktrees/<agent>-<id>
   if [ -n "$(git status --porcelain)" ]; then
     echo "WARNING: Uncommitted changes in worktree"
     # Ask user to confirm or abort
   fi

   # Merge branch
   git checkout main
   git merge --no-ff conductor/<agent>-<id> -m "conductor(cleanup): Merge orphaned worktree <agent>-<id>"

   # Remove worktree
   git worktree remove .worktrees/<agent>-<id>

   # Delete branch
   git branch -d conductor/<agent>-<id>
   ```

5. **If no**: Skip that worktree

6. **Report**: "Cleaned up N worktrees."

---

## Integration with /implement

After retrieving background implementer results:

1. **Verify plan.md reflects completed tasks**
2. **Check if phase is complete**
3. **Suggest next action**:
   - Continue implementation (more tasks)
   - Run phase verification (phase complete)
   - Check status (review progress)

---

## Safety Notes

- Always check for uncommitted changes before removing worktrees
- Use `--no-ff` merge to preserve branch history for audit trail
- Confirm with user before destructive operations
- Background agents may fail - always check status before cleanup
