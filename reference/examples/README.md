# Examples

> Complete implementation examples for Conductor on Claude Code

This directory contains working examples of all plugin components.

## Directory Structure

```
examples/
├── commands/           # Slash command implementations
│   ├── setup.md       # /conductor-okf:setup
│   ├── new-track.md   # /conductor-okf:new-track
│   ├── implement.md   # /conductor-okf:implement
│   ├── status.md      # /conductor-okf:status
│   └── revert.md      # /conductor-okf:revert
├── agents/            # Specialized subagent definitions
│   ├── planner.md      # Planning & specs
│   └── implementer.md  # Task execution
├── skills/            # Auto-discovered capabilities
│   ├── context-awareness/
│   │   └── SKILL.md   # Project context loading
│   └── tdd-workflow/
│       └── SKILL.md   # TDD methodology
└── hooks/
    └── hooks.json     # Event-driven automations
```

## Commands

| Command                | File                                    | Description                    |
| ---------------------- | --------------------------------------- | ------------------------------ |
| `/conductor-okf:setup`     | [setup.md](./commands/setup.md)         | Initialize project environment |
| `/conductor-okf:new-track`  | [new-track.md](./commands/new-track.md) | Create feature/bug track       |
| `/conductor-okf:implement` | [implement.md](./commands/implement.md) | Execute tasks with TDD         |
| `/conductor-okf:status`    | [status.md](./commands/status.md)       | Display progress report        |
| `/conductor-okf:revert`    | [revert.md](./commands/revert.md)       | Git-aware rollback             |

## Agents

| Agent       | File                                      | Purpose                  |
| ----------- | ----------------------------------------- | ------------------------ |
| Planner     | [planner.md](./agents/planner.md)         | Generate specs and plans |
| Implementer | [implementer.md](./agents/implementer.md) | Execute TDD workflow     |

## Skills

| Skill             | Directory                                         | Auto-Activates When           |
| ----------------- | ------------------------------------------------- | ----------------------------- |
| Context Awareness | [context-awareness/](./skills/context-awareness/) | `conductor/` directory exists |
| TDD Workflow      | [tdd-workflow/](./skills/tdd-workflow/)           | Implementing features         |

## Hooks

See [hooks.json](./hooks/hooks.json) for event-driven automations:

- **SessionStart**: Show track count on session start
- **PostToolUse**: Detect plan.md modifications
- **PreToolUse**: Warn when modifying core configs
- **Stop**: Remind about in-progress work

## Usage

To use these examples:

1. Copy the entire `examples/` structure to your plugin directory
2. Rename to match plugin structure:
   ```
   examples/commands/ → your-plugin/commands/
   examples/agents/ → your-plugin/agents/
   examples/skills/ → your-plugin/skills/
   examples/hooks/ → your-plugin/hooks/
   ```
3. Create plugin manifest (`.claude-plugin/plugin.json`)
4. Install plugin in Claude Code

## Key Patterns

### Command Frontmatter

```markdown
---
description: Brief description for help text
argument-hint: [optional] [arguments]
allowed-tools: Read, Write, Bash
model: inherit
---
```

### Agent Frontmatter

```markdown
---
name: agent-name
description: When to use this agent
tools: Read, Write, Glob
model: sonnet
---
```

### Skill Frontmatter

```yaml
---
name: skill-name
description: Auto-discovery trigger conditions
allowed-tools: Read, Glob
---
```

### Hook Structure

```json
{
  "EventName": [
    {
      "matcher": "pattern",
      "hooks": [
        {
          "type": "command",
          "command": "bash command",
          "timeout": 5
        }
      ]
    }
  ]
}
```
