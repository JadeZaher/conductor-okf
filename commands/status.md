---
description: Display current progress of Conductor-managed project
allowed-tools: Read, Glob, Bash
model: claude-haiku-4-5-20251001
---

# Conductor Status

Display comprehensive project status overview.

## Pre-flight Check

1. Check `conductor/tracks.md` exists. If not: "Run `/conductor-okf:setup` to initialize."
2. If empty: "No tracks. Create one with `/conductor-okf:new-track`."

## Gather Data

### Step 1: Read Core Files (Parallel)

**Use parallel Read tool calls for:**
- `conductor/product.md` - Project name and context
- `conductor/tracks.md` - Track list and status

### Step 2: Read All Track Plans (Parallel)

After listing track directories with `ls conductor/tracks/`, **issue parallel Read calls for all track `plan.md` files:**
- `conductor/tracks/<track1>/plan.md`
- `conductor/tracks/<track2>/plan.md`
- (etc. for all tracks)

For each plan.md, count:
- `[ ]` pending tasks
- `[~]` in-progress tasks
- `[x]` completed tasks
- Identify current phase and task

> **Performance note:** Reading all track plans in parallel significantly speeds up status reporting for projects with multiple tracks.

## Generate Report

Display status with:
- Project name and timestamp
- Tracks overview: Completed ✅ / In Progress 🔄 / Pending ⏳ counts
- Current focus: Track, Phase, Task (all `[~]` items)
- Progress bar: `[████░░░░] X%` (completed/total tasks)
- Track details: Tree view with status indicators per phase/task
- Next actions: Upcoming 2-3 tasks
- Call to action: "Run /conductor-okf:implement to continue"

### If Blockers

Show section with blocked tasks and reasons (from plan.md BLOCKED markers).

### If All Complete

Show celebration: "🎉 ALL TRACKS COMPLETE! Create new track with /conductor-okf:new-track"

### Summary Stats

- Total phases/tasks across all tracks
- Overall completion percentage
