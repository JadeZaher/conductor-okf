# Plugin Architecture

> Proposed structure for Conductor on Claude Code

## Directory Structure

```
conductor-claude/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest
├── commands/
│   ├── setup.md                       # /conductor-okf:setup
│   ├── new-track.md                   # /conductor-okf:new-track
│   ├── implement.md                   # /conductor-okf:implement
│   ├── status.md                      # /conductor-okf:status
│   └── revert.md                      # /conductor-okf:revert
├── agents/
│   ├── planner.md                     # Planning/spec generation
│   ├── implementer.md                 # Task execution
│   └── reviewer.md                    # Verification/checkpoints
├── skills/
│   ├── context-awareness/
│   │   └── SKILL.md                   # Auto-loads conductor/ context
│   ├── tdd-workflow/
│   │   └── SKILL.md                   # TDD guidance
│   └── code-styleguides/
│       └── SKILL.md                   # Style guide skill (reads from project)
├── hooks/
│   └── hooks.json                     # Event-driven automations
├── templates/
│   ├── workflow.md                    # Default workflow template
│   └── code-styleguides/              # Language-specific style templates
│       ├── typescript.md
│       ├── python.md
│       ├── go.md
│       ├── javascript.md
│       └── html-css.md
├── LICENSE
└── README.md
```

## Plugin Manifest

**`.claude-plugin/plugin.json`**

```json
{
  "name": "conductor",
  "version": "1.0.0",
  "description": "Context-Driven Development framework - plan before you build",
  "author": {
    "name": "Your Name",
    "email": "you@example.com",
    "url": "https://github.com/yourname"
  },
  "homepage": "https://github.com/yourname/conductor-claude",
  "repository": "https://github.com/yourname/conductor-claude",
  "license": "Apache-2.0",
  "keywords": [
    "planning",
    "tdd",
    "workflow",
    "context-driven",
    "project-management",
    "specifications"
  ],
  "commands": ["./commands/"],
  "agents": "./agents/",
  "skills": "./skills/",
  "hooks": "./hooks/hooks.json"
}
```

## Commands

### Command Frontmatter Options

| Field           | Type   | Description                       |
| --------------- | ------ | --------------------------------- |
| `description`   | string | Brief description (shown in help) |
| `argument-hint` | string | Expected arguments format         |
| `allowed-tools` | string | Comma-separated tool list         |
| `model`         | string | Specific model to use             |

### `/conductor-okf:setup`

```markdown
---
description: Initialize Conductor environment for context-driven development
argument-hint: [project-type]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Conductor Setup

Initialize the Conductor context-driven development environment.

## Pre-flight

Check for existing conductor/ directory:

- If exists with setup_state.json, offer to resume
- If exists without state, warn about overwrite

## Phase 1: Project Discovery

...
```

### `/conductor-okf:new-track`

```markdown
---
description: Create a new feature or bug track with spec and plan
argument-hint: [description]
allowed-tools: Read, Write, Glob, Task
---

# New Track

Create a new track (feature, bug, chore) with specification and plan.

## Arguments

Track description: $ARGUMENTS

If no description provided, ask user.
...
```

### `/conductor-okf:implement`

```markdown
---
description: Execute tasks from the current track following TDD workflow
argument-hint: [track-name]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task
---

# Implement Track

Execute implementation following the project workflow.
...
```

### `/conductor-okf:status`

```markdown
---
description: Display current progress of Conductor-managed project
allowed-tools: Read, Glob, Bash
---

# Project Status

Generate a comprehensive status report.
...
```

### `/conductor-okf:revert`

```markdown
---
description: Revert a track, phase, or task using git-aware rollback
argument-hint: [target]
allowed-tools: Read, Bash, Glob
---

# Smart Revert

Git-aware revert that understands logical work units.
...
```

## Agents

### planner

**Purpose:** Generate specifications and implementation plans.

```markdown
---
name: planner
description: Specialist for generating specifications and implementation plans. Use when creating new tracks, writing specs, or breaking down features into tasks.
tools: Read, Write, Glob, Grep
model: inherit
---

You are the Conductor Planning Agent...
```

**Responsibilities:**

- Requirements analysis
- Specification writing (spec.md)
- Task decomposition (plan.md)
- TDD task structuring

### implementer

**Purpose:** Execute tasks following TDD workflow.

```markdown
---
name: implementer
description: Specialist for executing implementation tasks following TDD workflow. Use when implementing features, fixing bugs, or working through plan.md tasks.
tools: Read, Write, Edit, Bash, Glob, Grep
model: inherit
---

You are the Conductor Implementation Agent...
```

**Responsibilities:**

- TDD cycle execution (Red → Green → Refactor)
- Plan progress updates
- Git commits with proper messages
- Quality gate enforcement

### reviewer

**Purpose:** Handle verification and checkpoints.

```markdown
---
name: reviewer
description: Specialist for phase verification and checkpoint creation. Use when completing phases, running verification protocols, or creating checkpoint commits.
tools: Read, Bash, Glob, Grep
model: inherit
---

You are the Conductor Review Agent...
```

**Responsibilities:**

- Test coverage verification
- Manual verification plan generation
- Checkpoint commit creation
- Git notes for audit trail

## Skills

### context-awareness

**Auto-discovers** when `conductor/` directory exists.

```yaml
---
name: conductor-context
description: Auto-load Conductor project context when conductor/ directory exists. Use for any development task in a Conductor-managed project.
allowed-tools: Read, Glob
---

# Conductor Context Awareness

When you detect a `conductor/` directory, automatically consider:

1. `conductor/product.md` - What we're building
2. `conductor/tech-stack.md` - Technologies in use
3. `conductor/workflow.md` - Development methodology
4. `conductor/tracks.md` - Current project status

## Key Context Points

- **Coverage Target**: From workflow.md
- **Commit Strategy**: From workflow.md
- **Style Guides**: From conductor/code_styleguides/
```

### tdd-workflow

**Auto-discovers** when working on implementation tasks.

```yaml
---
name: tdd-workflow
description: Test-Driven Development guidance. Use when writing code, implementing features, or fixing bugs in projects that follow TDD methodology.
---
# TDD Workflow Skill

## The TDD Cycle

### 1. Red Phase
- Write a failing test FIRST
- Run tests, confirm FAILURE

### 2. Green Phase
- Write MINIMUM code to pass
- Run tests, confirm PASS

### 3. Refactor Phase
- Clean up while tests pass
- Run tests after each change
```

### code-styleguides

**Auto-discovers** when writing code in specific languages. Reads styleguides from `conductor/code_styleguides/` in the project.

```yaml
---
name: code-styleguides
description: Language-specific code style guidelines. Use when writing TypeScript, Python, Go, JavaScript, or HTML/CSS code.
---
# Code Style Guides

Reads from conductor/code_styleguides/ which are copied during /conductor-okf:setup.
See templates/code-styleguides/ for available language styleguides.
```

## Hooks

**`hooks/hooks.json`**

```json
{
  "SessionStart": [
    {
      "matcher": "startup|resume",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -d conductor ] && [ -f conductor/tracks.md ]; then in_progress=$(grep -c '\\[~\\]' conductor/tracks.md 2>/dev/null || echo 0); pending=$(grep -c '\\[ \\]' conductor/tracks.md 2>/dev/null || echo 0); echo \"{\\\"systemMessage\\\": \\\"Conductor: ${in_progress} in-progress, ${pending} pending tracks\\\"}\"; fi",
          "timeout": 5
        }
      ]
    }
  ],
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "command",
          "command": "jq -r '.tool_input.file_path // empty' | { read fp; if echo \"$fp\" | grep -q 'conductor/tracks/.*/plan.md'; then echo 'Plan updated'; fi; }",
          "timeout": 5
        }
      ]
    }
  ],
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -f conductor/tracks.md ] && grep -q '\\[~\\]' conductor/tracks.md 2>/dev/null; then echo '{\"systemMessage\": \"Reminder: You have in-progress tracks. Run /conductor-okf:status for details.\"}'; fi",
          "timeout": 5
        }
      ]
    }
  ]
}
```

### Hook Purposes

| Event          | Purpose                           |
| -------------- | --------------------------------- |
| `SessionStart` | Show track count on session start |
| `PostToolUse`  | Detect plan.md modifications      |
| `Stop`         | Remind about in-progress work     |

## Templates

Templates are copied during `/conductor-okf:setup`:

### workflow.md

- Guiding principles
- Task lifecycle (TDD)
- Phase completion protocol
- Quality gates
- Commit guidelines

### product.md

- Product vision
- Target users
- Goals and features
- Success metrics

### product-guidelines.md

- Brand voice
- Design standards
- Communication style

## Environment Variables

Plugins can use these variables:

| Variable                | Description                       |
| ----------------------- | --------------------------------- |
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to plugin directory |
| `${CLAUDE_PROJECT_DIR}` | Project root directory            |

Example in hooks:

```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

## Installation

### User-wide (recommended)

Add to `~/.claude/settings.json`:

```json
{
  "plugins": ["https://github.com/yourname/conductor-claude"]
}
```

### Project-specific

Add to `.claude/settings.json`:

```json
{
  "plugins": ["./path/to/conductor-claude"]
}
```

## Namespacing

All commands are namespaced with the plugin name:

- `/conductor-okf:setup`
- `/conductor-okf:new-track`
- `/conductor-okf:implement`
- `/conductor-okf:status`
- `/conductor-okf:revert`

This prevents conflicts with other plugins.
