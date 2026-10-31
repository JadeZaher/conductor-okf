# Claude Conductor

Context-Driven Development framework for Claude Code.

> Inspired by [Conductor for Gemini CLI](https://github.com/gemini-cli-extensions/conductor)

## Overview

Conductor enables **Context-Driven Development** - a methodology where context is treated as a managed artifact alongside code. It provides:

- **Pre-implementation Planning**: Generate specs and plans before coding
- **Test-Driven Development**: Enforced Red-Green-Refactor cycles
- **Context Management**: Maintain style guides, tech stack, product goals
- **Iterative Safety**: Review plans before execution with checkpoints
- **Intelligent Reversion**: Git-aware rollback of logical work units
- **Session Handoff**: A `conductor/RUNBOOK.md` that survives compaction, crashes, and the end of a session — with a recorded review verdict per phase
- **Parallel Orchestration**: Conflict-free file partitions computed at planning time and consumed by a pre-grilled fan-out

## Installation

### From GitHub

```
/plugin install conductor-okf --marketplace JadeZaher/conductor-okf
```

### From a zip

`scripts/build-plugin-zip.py` writes `dist/conductor-okf-<version>.zip` with the
plugin root at the **archive root** — no wrapper folder, because Claude Code
extracts the archive and treats the extracted directory as the plugin root.

```bash
python scripts/build-plugin-zip.py          # vendor + validate + package
python scripts/build-plugin-zip.py --strict # fail the build on validator warnings
```

> The build script ships in the source tree, not in the archive it produces —
> a distributed zip contains the plugin only.

**One session only** — the zip is extracted to a session directory and discarded,
so nothing is installed and nothing is left behind:

```bash
claude --plugin-dir dist/conductor-okf-1.3.0.zip
```

**One session, from a URL:**

```bash
claude --plugin-url https://example.com/conductor-okf-1.3.0.zip
```

**Installed permanently** — point a marketplace entry at the hosted zip, then
install from it. Any static host works; the archive is fetched and cached:

```json
{
  "name": "conductor-okf",
  "version": "1.3.0",
  "source": {
    "source": "url",
    "url": "https://example.com/conductor-okf-1.3.0.zip"
  }
}
```

```bash
/plugin marketplace add https://example.com/my-marketplace.json
/plugin install conductor-okf@my-marketplace
```

### From a local directory

```bash
claude --plugin-dir /path/to/conductor-okf     # one session
claude plugin validate /path/to/conductor-okf  # check the manifest first
```

For a permanent local install, add the containing directory as a marketplace
(it needs a `.claude-plugin/marketplace.json` listing `"source": "./conductor-okf"`):

```bash
/plugin marketplace add ~/.claude/plugins/marketplaces/local
/plugin install conductor-okf@local
```

### Verify

```bash
# In a session with the plugin loaded, type the namespace prefix:
/conductor-okf:
```

You should see **8** commands — the six conductor commands plus `handoff` and
`passoff` — and `claude plugin details conductor-okf` should list the
`handoff`, `slice`, `tdd-workflow`, `code-styleguides`, and `context-awareness`
skills.

## Commands

### `/conductor-okf:setup`

Initialize Conductor environment for a project.

**Usage:**
```
/conductor-okf:setup [brownfield|greenfield]
```

**Arguments:**
- `brownfield` - Skip detection, treat as existing project
- `greenfield` - Skip detection, treat as new project

**What it does:**
1. Detects project maturity (brownfield vs greenfield)
2. Creates `conductor/` directory with context files
3. Interactively generates `product.md`, `tech-stack.md`, `workflow.md`
4. Creates first track with specification and plan

**Example:**
```
> /conductor-okf:setup

Existing project detected. I'll analyze it to understand the current state.
Found: package.json, src/, .git

What do you want to build? [describe your first feature]
```

---

### `/conductor-okf:new-track`

Create a new feature, bug, or chore track.

**Usage:**
```
/conductor-okf:new-track [description]
```

**Arguments:**
- `description` - Brief description of the work (optional, will prompt if not provided)

**What it does:**
1. Infers track type (feature/bug/chore) from description
2. Delegates to planner agent for interactive Q&A
3. Generates `spec.md` with requirements
4. Generates `plan.md` with TDD-structured tasks
5. Creates track directory and updates `tracks.md`

**Example:**
```
> /conductor-okf:new-track Add user authentication with OAuth

Creating feature track...
I have some questions to clarify the requirements:
1. Which OAuth providers should be supported?
2. Should we include "remember me" functionality?
...
```

---

### `/conductor-okf:implement`

Execute tasks following TDD workflow.

**Usage:**
```
/conductor-okf:implement [<track-name>] [--all]
```

**Arguments:**
- `<track-name>` - Partial name to select specific track (optional, selects next incomplete)
- `--all` - Run all phases instead of one at a time

**What it does:**
1. Selects next incomplete track (or specified track)
2. **Single-phase mode (default)**: Shows interactive phase selection, implements one phase at a time
3. **All-phases mode (`--all`)**: Implements all remaining phases continuously
4. Delegates each task to implementer agent
5. Follows TDD cycle: Red → Green → Refactor
6. Creates atomic commits with git notes
7. Triggers phase verification at phase boundaries
8. Creates checkpoints with reviewer agent

**Examples:**
```
> /conductor-okf:implement

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PHASE SELECTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Track: User Authentication

  [x] Phase 1: Setup                    [complete]
  [ ] Phase 2: Backend API              [in progress - 2/5 tasks]
  [ ] Phase 3: Frontend Integration     [pending]

  Which phase would you like to implement?

  A) Phase 2: Backend API (recommended)
  B) Phase 3: Frontend Integration
  C) All remaining phases
```

```
> /conductor-okf:implement --all

Running all remaining phases for track 'User Authentication'.
Current task: Implement JWT validation

[RED] Writing failing test...
[GREEN] Implementing to pass...
[REFACTOR] Cleaning up...

Task complete. Commit: feat(auth): Implement JWT validation [a1b2c3d]
```

---

### `/conductor-okf:status`

Display project progress report.

**Usage:**
```
/conductor-okf:status
```

**What it does:**
1. Parses all tracks and plans
2. Calculates completion percentages
3. Identifies current focus (in-progress track/phase/task)
4. Lists any blockers
5. Shows next actions

**Example output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  CONDUCTOR STATUS REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CURRENT FOCUS
───────────────────────────────────────────────────
  Track: User Authentication
  Phase: Backend API [~]
  Task:  Implement JWT validation [~]

PROGRESS
───────────────────────────────────────────────────
  [████████████░░░░░░░░] 60%  (12/20 tasks)
```

---

### `/conductor-okf:agents`

Manage background Conductor agents and git worktrees.

**Usage:**
```
/conductor-okf:agents [status|results|worktrees|cleanup]
```

**Arguments:**
- `status` (default) - Check background agent status
- `results` - Retrieve results from completed agents
- `worktrees` - List git worktrees used for isolation
- `cleanup` - Remove orphaned worktrees

**What it does:**
1. Lists running background agents (implementer, planner, reviewer)
2. Shows worktree isolation status
3. Retrieves completed agent results
4. Merges and cleans up worktree branches

**Example:**
```
> /conductor-okf:agents

Background Agents:
  [RUNNING] implementer - Task: Add user validation (worktree: .worktrees/implementer-1735075200)

Worktrees:
  .worktrees/implementer-1735075200 → conductor/implementer-1735075200
```

```
> /conductor-okf:agents results

Implementer completed:
  ✓ 3 tasks completed
  ✓ 4 commits created
  ✓ Checkpoint: a1b2c3d

Merging worktree back to main...
```

---

### `/conductor-okf:revert`

Git-aware rollback of tracks, phases, or tasks.

**Usage:**
```
/conductor-okf:revert [track|phase|task] [name]
```

**Arguments:**
- `track <name>` - Revert entire track
- `phase <name>` - Revert specific phase
- `task <name>` - Revert specific task

**What it does:**
1. Identifies all commits associated with the target
2. Finds both implementation and plan update commits
3. Presents revert plan for confirmation
4. Executes `git revert` in correct order
5. Handles conflicts with user guidance
6. Updates plan.md status markers

**Example:**
```
> /conductor-okf:revert phase "Backend API"

I have analyzed the git history. Here is the revert plan:

Commits to revert (newest first):
  1. b2c3d4e - conductor(plan): Complete 'Add rate limiting'
  2. a1b2c3d - feat(api): Add rate limiting to endpoints

Do you want to proceed? [Y/n]
```

## Runbook Workflow

Conductor's context files (`product.md`, `tech-stack.md`, `workflow.md`,
`tracks.md`) are the **foundation**. `conductor/RUNBOOK.md` is the **summary** on
top of them: current state, settled decisions, stated assumptions, the review
ledger, and an executable continuation plan. Two skills own that layer.

### `handoff` — serial continuation

`skills/handoff/` packages the session so a fresh one resumes without
re-discovery. Nothing auto-loads the runbook; reading and writing it are always
explicit acts.

| Mode | Invocation | Lands |
|---|---|---|
| **inline** (default) | "handoff", "passoff", "where were we" | A pasteable packet in chat. Writes nothing. |
| **durable** | `/conductor-okf:handoff` | `conductor/RUNBOOK.md` |
| **rolling** | `/conductor-okf:handoff rolling` | Incremental checkpoints *during* execution |
| **resume** | `/conductor-okf:handoff resume` | Reads the runbook back and verifies it against `git status` |

`/conductor-okf:passoff` is an alias — same skill, same behavior.

**Target rule** (`skills/handoff/references/runbook.md` is authoritative):
`conductor/` exists → `conductor/RUNBOOK.md`; no `conductor/` → create the
directory and write it there; a legacy `.omc/RUNBOOK.md` → migrate and delete.
Creating the directory is a `mkdir` plus one file — **never**
`/conductor-okf:setup`, which is a five-round interview that writes ~8 files.

### `slice` — parallel fan-out

`skills/slice/` is the sibling for work that goes *wide* instead of *forward*:
it cuts a task into agent-sized units with pre-declared, conflict-free file
boundaries, assigns a model tier per slice, derives waves from `depends_on`, and
runs a pre-launch grilling gate before any agent is spawned.

Two laws it enforces: two agents writing one file collide, so make collisions
impossible by construction; and slice agents *sketch* while the orchestrator
*inks* — all building, testing, and linting happens once, serially, at the wave
join.

### Prebaked partitions — the link back to tracks

`/conductor-okf:new-track` already reads the codebase to write the spec, so it
fills `conductor/tracks/<id>/metadata.json` → `partitions` from that same
analysis. `slice` reads it instead of starting from a blank page:

```json
"partitions": {
  "computed_at_commit": "a1b2c3d",
  "confidence": "planned",
  "slices": [{"id": "s1", "task": "Extract the quest store",
              "owns": ["src/stores/quest.ts"], "model": "sonnet", "depends_on": []}],
  "shared_writes": [{"path": "src/registry.ts", "owner": "s3", "reason": "s3 serializes"}]
}
```

A partition is a **hypothesis stamped with a commit**, never a fact. If `HEAD`
has moved, re-verify every `owns` list with a real grep. `confidence: "none"` is
always better than a guess — a wrong partition looks authoritative and sends two
agents at the same file.

### Shared references

Both skills read from `skills/handoff/references/`, so each concept is defined
once:

| Reference | Defines |
|---|---|
| [`runbook.md`](skills/handoff/references/runbook.md) | Target rule, file format, clipping, resume protocol |
| [`grilling.md`](skills/handoff/references/grilling.md) | The one-round question gate — flip test, ranking, recording |
| [`rolling.md`](skills/handoff/references/rolling.md) | Checkpoints, phase-boundary review verdicts, track markers |
| [`invocations.md`](skills/handoff/references/invocations.md) | The weight ladder: inline → skill → agent → fan-out → ultra |
| [`partitions.md`](skills/handoff/references/partitions.md) | Partition schema and the consumer contract |

### The review ledger

Every phase boundary gets an **adversarial review whose verdict is recorded** in
runbook §2 — run in a context separate from the one that wrote the code, and
prompted to refute rather than confirm. A phase with no verdict is *unreviewed*,
not done, however finished the code looks. Verdicts are `PASS`,
`PASS-with-deferred`, or `BLOCKED`; deferred findings move to §4 Assumptions with
their reversal cost. Every code checkpoint loads
`conductor-okf:code-styleguides` so findings cite this project's rules rather
than generic taste.

### Maintaining the vendored copy

These skills are vendored into the plugin from a canonical source (by default
`~/.claude/skills/` and `~/.claude/commands/`). Edit the canonical copy, then
re-run the build — it re-copies and rewrites user-level paths to
`${CLAUDE_PLUGIN_ROOT}` and bare slash commands to namespaced ones:

```bash
python scripts/build-plugin-zip.py
```

Hand-editing `skills/handoff/`, `skills/slice/`, `commands/handoff.md`, or
`commands/passoff.md` inside the plugin works until the next build overwrites it.
To build from a different canonical source, pass `--claude-home <path>`; to
package the tree as-is, pass `--skip-vendor`.

## Configuration

### Workflow Customization

Edit `conductor/workflow.md` after setup to customize:

| Setting | Default | Description |
|---------|---------|-------------|
| Coverage Target | 80% | Minimum test coverage percentage |
| Commit Strategy | Per Task | When to commit (per-task or per-phase) |
| Git Notes | Enabled | Attach summaries to commits |
| Mobile Testing | If Applicable | Require mobile verification |
| Parallel Agents | worktree | Strategy for parallel agent execution |

### Quality Gates

The workflow enforces these checks before task completion:

- All tests pass
- Code coverage meets target (>80%)
- Code follows style guides
- No linting errors
- Type safety enforced
- Documentation updated

### Code Styleguides

Language-specific style guides are provided in `templates/code-styleguides/`:

- TypeScript
- Python
- Go
- JavaScript
- HTML/CSS

During `/conductor-okf:setup`, selected styleguides are copied to `conductor/code_styleguides/` based on your tech stack. The `code-styleguides` skill auto-activates when writing code, reading from your project's styleguide files.

## Background Execution & Parallel Agents

Conductor supports running agents in the background for long-running tasks and parallel workflows.

### When to Use Background Execution

```bash
# Run implementer in background for long task
> I want to implement user validation. Can you run that in the background while I work on something else?

# Run multiple tasks in parallel
> Start implementing authentication in the background, and I'll work on the frontend
```

### Parallel Agent Strategies

Configure in `conductor/workflow.md`:

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **worktree** (recommended) | Each background agent works in isolated git worktree | Parallel development without conflicts |
| **sequential** | Background agents queue, run one at a time | Safe fallback when worktrees not desired |
| **unsafe** | No isolation, agents work on same branch | Not recommended (manual conflict resolution) |

### How Worktree Isolation Works

When `parallel_agents: worktree`:

1. **Agent spawns**: Creates `.worktrees/implementer-<id>/` with separate branch
2. **Isolated work**: All file changes happen in worktree directory
3. **On completion**: Branch merges back to main automatically
4. **Cleanup**: Worktree and branch removed after merge

**Benefits:**
- ✅ No git conflicts during parallel execution
- ✅ Each agent has independent workspace
- ✅ Clean merge history with `--no-ff`
- ✅ Automatic cleanup on success

**Example:**
```bash
> /conductor-okf:agents

Background Agents:
  [RUNNING] implementer - worktree: .worktrees/implementer-1735075200

# Agent completes...

> /conductor-okf:agents results

Implementer completed. Merging worktree...
✓ Merged conductor/implementer-1735075200 → main
✓ Removed worktree
```

### Skills Auto-Loading

Agents now preload skills at startup for faster context loading:

| Agent | Preloaded Skills |
|-------|-----------------|
| **planner** | context-awareness |
| **implementer** | context-awareness, tdd-workflow, code-styleguides |
| **reviewer** | context-awareness |

This ensures agents have immediate access to:
- Project context (product.md, tech-stack.md, workflow.md)
- TDD methodology (Red-Green-Refactor cycle)
- Language-specific style guidelines

## Project Structure

When initialized, Conductor creates:

```
your-project/
└── conductor/
    ├── product.md           # Product vision and goals
    ├── product-guidelines.md # Brand and design standards
    ├── tech-stack.md        # Technology choices
    ├── workflow.md          # Development methodology
    ├── setup_state.json     # Resume capability state
    ├── tracks.md            # Track overview
    ├── RUNBOOK.md           # Session state, decisions, review ledger (handoff skill)
    └── tracks/
        └── <track_id>/
            ├── spec.md      # Requirements specification
            ├── plan.md      # Implementation plan
            └── metadata.json # Machine source of truth + `partitions`
```

## Troubleshooting

### Common Errors

**"Conductor is not set up"**
```
Run /conductor-okf:setup first to initialize the conductor/ directory.
```

**"No tracks found"**
```
Run /conductor-okf:new-track to create your first track.
```

**"You have uncommitted changes"**
```
Commit or stash your changes before running /conductor-okf:revert.
```

### Prerequisites

Before using Conductor:

1. **Git repository**: Project must be a git repo (or `/conductor-okf:setup` will init one)
2. **Claude Code**: Version with plugin support
3. **Test framework**: Project should have testing set up for TDD workflow

### Debug Mode

For verbose output showing hook execution:

```bash
claude --verbose --plugin-dir /path/to/conductor
```

Check hook execution logs:
```
~/.claude/debug/
```

### Resuming Interrupted Setup

If setup is interrupted, it can be resumed:

```bash
# Setup saves state in conductor/setup_state.json
# Re-running setup will offer to resume from last successful step
/conductor-okf:setup
```

### Session Resumption

Conductor automatically resumes work when starting a new session:

1. **SessionStart hook** detects conductor project and shows current status
2. **plan.md status markers** track progress (`[ ]` pending, `[~]` in-progress, `[x]` complete)
3. **Run `/conductor-okf:implement`** to continue from where you left off

No special handoff files needed - plan.md is the single source of truth.

## Architecture

Conductor uses Claude Code's plugin system:

| Component | Purpose |
|-----------|---------|
| **Commands** (`/conductor-okf:*`) | User-invoked actions |
| **Agents** | Specialized subagents with preloaded skills (planner, implementer, reviewer) |
| **Skills** | Auto-loaded capabilities via frontmatter (TDD, styleguides, context, `handoff`, `slice`) |
| **Hooks** | Event-driven automation (context loading, change tracking) |
| **Worktrees** | Git isolation for parallel agent execution |

## Documentation

See the `reference/` directory for detailed documentation:

- [Implementation Blueprint](reference/implementation-blueprint.md)
- [Plugin Architecture](reference/plugin-architecture.md)
- [Claude Code Mapping](reference/claude-code-mapping.md)
- [Conductor Analysis](reference/conductor-analysis.md)
- [OKF Format](reference/okf-format.md) — the frontmatter convention on every context and track file

The runbook layer documents itself in `skills/handoff/references/` — see [Shared references](#shared-references) above.

## License

Apache-2.0
