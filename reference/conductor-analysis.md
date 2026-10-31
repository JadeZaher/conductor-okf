# Conductor Architecture Analysis

> Deep dive into the Gemini CLI extension

**Source Repository**: [github.com/gemini-cli-extensions/conductor](https://github.com/gemini-cli-extensions/conductor)
**License**: Apache License 2.0
**Version Analyzed**: 0.1.1

## Core Philosophy

Conductor enables **Context-Driven Development** - a methodology where context is treated as a managed artifact alongside code. This creates a single source of truth that provides persistent project awareness to AI agents.

The guiding principle: **"Measure twice, code once."**

## Feature Overview

| Feature | Description |
|---------|-------------|
| Pre-implementation Planning | Generate specs and plans before coding |
| Context Management | Maintain style guides, tech stack, product goals |
| Iterative Safety | Review plans before execution |
| Team Collaboration | Share project-level context across team |
| Project Flexibility | Supports greenfield and brownfield projects |
| Intelligent Reversion | Git-aware rollback of logical work units |

## Command Structure

Conductor provides 5 core commands:

| Command | Purpose | Generated Artifacts |
|---------|---------|---------------------|
| `/conductor-okf:setup` | One-time project initialization | `product.md`, `tech-stack.md`, `workflow.md`, `tracks.md` |
| `/conductor-okf:new-track` | Create feature/bug work unit | `spec.md`, `plan.md`, `metadata.json` |
| `/conductor-okf:implement` | Execute planned tasks | Updates to plans, commits |
| `/conductor-okf:status` | Display progress overview | None (read-only) |
| `/conductor-okf:revert` | Undo work logically | Git reverts |

## File Structure

```
project/
├── conductor/
│   ├── product.md              # Product vision, users, goals
│   ├── product-guidelines.md   # Brand voice, design standards
│   ├── tech-stack.md           # Technologies, frameworks
│   ├── workflow.md             # TDD settings, commit strategy
│   ├── setup_state.json        # Resume capability state
│   ├── tracks.md               # Master track list
│   ├── code_styleguides/       # Language-specific style guides
│   │   ├── typescript.md
│   │   ├── python.md
│   │   └── ...
│   └── tracks/
│       └── <track_id>/
│           ├── spec.md         # Requirements specification
│           ├── plan.md         # Implementation plan
│           └── metadata.json   # Track metadata
└── ...
```

## Hierarchical Work Model

```
Tracks (features/bugs)
  └── Phases (logical groupings)
       └── Tasks (actionable items)
            └── Sub-tasks (TDD cycles)
```

### Task Status Markers

- `[ ]` - Pending
- `[~]` - In Progress
- `[x]` - Completed

## Command Implementation (TOML Format)

Conductor commands are defined in TOML files:

```toml
description = "Brief description shown in help"
prompt = """
## 1.0 SYSTEM DIRECTIVE
You are an AI agent...

CRITICAL: You must validate the success of every tool call...

## 2.0 PHASE 1: SECTION NAME
**PROTOCOL: Follow this sequence precisely.**

### 2.1 Step Name
1. First action
2. Second action
...
"""
```

### Key Prompt Patterns

1. **System Directives**: Clear role definition and constraints
2. **Protocols**: Numbered sequences with explicit steps
3. **Critical Markers**: Emphasized requirements using `CRITICAL:`
4. **State Management**: Resume capability via JSON state files
5. **User Confirmation Loops**: Present draft → review → revise → approve
6. **Interactive Q&A**: Sequential questions with A/B/C/D/E options

## Workflow System

The `workflow.md` template defines:

### Guiding Principles
1. The Plan is the Source of Truth
2. The Tech Stack is Deliberate
3. Test-Driven Development
4. High Code Coverage (>80%)
5. User Experience First
6. Non-Interactive & CI-Aware

### Task Lifecycle
1. Select Task from plan.md
2. Mark In Progress (`[~]`)
3. Write Failing Tests (Red Phase)
4. Implement to Pass (Green Phase)
5. Refactor
6. Verify Coverage
7. Document Deviations
8. Commit Code Changes
9. Attach Git Notes
10. Update Plan with SHA
11. Commit Plan Update

### Phase Completion Protocol
1. Ensure test coverage for phase changes
2. Execute automated tests with debugging
3. Propose manual verification plan
4. Await user confirmation
5. Create checkpoint commit
6. Attach verification report as git note
7. Record checkpoint SHA in plan

## State Management

Conductor uses JSON for resumable state:

```json
{
  "last_successful_step": "2.3_tech_stack"
}
```

This enables:
- Resuming interrupted setup
- Tracking progress across sessions
- Idempotent operations

## Git Integration

Conductor is deeply Git-aware:

1. **Commits per task**: Atomic, traceable changes
2. **Git Notes**: Detailed task summaries attached to commits
3. **Checkpoint commits**: Phase completion markers
4. **Smart revert**: Understands tracks/phases/tasks, not just commits
5. **History analysis**: Finds related commits for logical reverts

## Interactive Patterns

### Question Format
```
Question text here? (Select all that apply)

A) Option A
B) Option B
C) Option C
D) Type your own answer
E) Auto-generate and continue
```

### Confirmation Loop
```
I've drafted the document. Please review:

[content]

A) Approve: The document is correct
B) Suggest Changes: Tell me what to modify
```

## Brownfield vs Greenfield Detection

### Brownfield Indicators
- `.git`, `.svn`, `.hg` directories
- Dirty git repository
- Dependency manifests (`package.json`, `requirements.txt`, etc.)
- Source directories with code (`src/`, `app/`, `lib/`)

### Greenfield Condition
- None of the above indicators found
- Directory empty or contains only documentation

## Extension Installation

```bash
gemini extensions install https://github.com/gemini-cli-extensions/conductor --auto-update
```

Templates are stored in:
```
~/.gemini/extensions/conductor/templates/
```
