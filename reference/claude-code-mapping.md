# Claude Code Extensibility Mapping

> How Conductor concepts translate to Claude Code primitives

## Claude Code Extensibility Overview

Claude Code provides six extensibility mechanisms:

| Mechanism       | Invocation            | Purpose                          |
| --------------- | --------------------- | -------------------------------- |
| **Skills**      | Auto-discovered       | Model-invoked capabilities       |
| **Agents**      | Delegated             | Specialized AI personalities     |
| **Plugins**     | Installed             | Distributable extension packages |
| **Commands**    | User-invoked (`/cmd`) | Reusable prompt templates        |
| **Hooks**       | Event-driven          | Automation at lifecycle points   |
| **MCP Servers** | Tool integration      | External service connections     |

## Mapping Table

| Conductor Concept | Claude Code Equivalent     | Advantages                                                    |
| ----------------- | -------------------------- | ------------------------------------------------------------- |
| TOML Commands     | **Slash Commands** (`.md`) | Richer frontmatter, argument placeholders, bash execution     |
| System Directives | **Agents/Subagents**       | Dedicated context windows, tool restrictions, model selection |
| Templates         | **Skills**                 | Auto-discovery, multi-file support, progressive loading       |
| State Files       | **Hooks** (SessionStart)   | Automatic context loading, event-driven updates               |
| Interactive Q&A   | **AskUserQuestion**        | Native UI support, multi-select, structured responses         |
| The whole thing   | **Plugin**                 | Versioned, distributable, namespaced package                  |

## Deep Dive: Each Mapping

### 1. Commands → Slash Commands

**Conductor (TOML):**

```toml
description = "Setup the project"
prompt = """
You are an AI agent...
"""
```

**Claude Code (Markdown):**

```markdown
---
description: Setup the project
argument-hint: [project-type]
allowed-tools: Read, Write, Bash
model: claude-sonnet-4-20250514
---

# Setup Command

You are an AI agent...

## Context

- Current directory: !`pwd`
- Git status: !`git status --porcelain`

## Arguments

Project type: $1 (or $ARGUMENTS for all)
```

**Advantages:**

- Frontmatter for metadata
- Argument placeholders (`$1`, `$ARGUMENTS`)
- Inline bash execution (`!`command``)
- File references (`@path/to/file`)
- Tool restrictions per command

### 2. System Directives → Agents

**Conductor approach:** Single massive prompt with everything inline.

**Claude Code approach:** Dedicated agents with separate context windows.

```markdown
---
name: planner
description: Specialist for specifications and plans
tools: Read, Write, Glob, Grep
model: sonnet
---

You are the Conductor Planning Agent...
```

**Advantages:**

- Separate context window (no pollution)
- Tool restrictions (read-only for exploration)
- Model selection (haiku for quick tasks, opus for complex)
- Reusable across commands

### 3. Templates → Skills

**Conductor:** Copies template files manually during setup.

**Claude Code:** Skills are auto-discovered based on context.

```yaml
---
name: tdd-workflow
description: Test-Driven Development guidance. Use when implementing features in TDD projects.
---
# TDD Workflow

## The Cycle
1. Red: Write failing test
2. Green: Minimum code to pass
3. Refactor: Clean up
```

**Advantages:**

- **Auto-discovery**: Claude uses skill when context matches
- Multi-file support (reference.md, examples.md)
- Progressive loading (only loads what's needed)
- No explicit invocation required

### 4. State Management → Hooks

**Conductor:** Manually reads/writes `setup_state.json`.

**Claude Code:** Hooks automate state handling.

```json
{
  "SessionStart": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "cat conductor/setup_state.json 2>/dev/null || echo '{}'"
        }
      ]
    }
  ]
}
```

**Advantages:**

- Automatic execution on events
- No manual context loading
- Can modify tool inputs/outputs
- Permission control (allow/deny/ask)

### 5. Interactive Q&A → AskUserQuestion

**Conductor:**

```
What is your target user?

A) Developers
B) End Users
C) Enterprise
D) Type your own
```

**Claude Code:**

```typescript
AskUserQuestion({
  questions: [
    {
      question: 'What is your target user?',
      header: 'Users',
      options: [
        { label: 'Developers', description: 'Technical users' },
        { label: 'End Users', description: 'Non-technical consumers' },
        { label: 'Enterprise', description: 'Business users' },
      ],
      multiSelect: true,
    },
  ],
});
```

**Advantages:**

- Native UI rendering
- Multi-select support
- Descriptions for each option
- Automatic "Other" option
- Structured response data

### 6. Extension → Plugin

**Conductor:** Git repository with TOML commands.

**Claude Code:** Structured plugin package.

```json
{
  "name": "conductor",
  "version": "1.0.0",
  "description": "Context-Driven Development",
  "commands": ["./commands/"],
  "agents": "./agents/",
  "skills": "./skills/",
  "hooks": "./hooks/hooks.json"
}
```

**Advantages:**

- Semantic versioning
- Namespaced commands (`/conductor-okf:setup`)
- Bundled agents, skills, hooks
- Environment variable expansion
- Shareable via npm or git

## Feature Comparison Matrix

| Feature           | Conductor   | Claude Code        |
| ----------------- | ----------- | ------------------ |
| Command format    | TOML        | Markdown           |
| Argument handling | `{{args}}`  | `$1`, `$ARGUMENTS` |
| Inline execution  | None        | `!`command``       |
| File references   | Manual read | `@path/to/file`    |
| Tool restrictions | None        | Per-command/agent  |
| Model selection   | None        | Per-command/agent  |
| Auto-discovery    | None        | Skills             |
| Event automation  | None        | Hooks              |
| Context isolation | None        | Subagents          |
| Distribution      | Git clone   | Plugin package     |

## Hook Events Available

| Event              | Use Case                             |
| ------------------ | ------------------------------------ |
| `SessionStart`     | Load conductor context automatically |
| `PreToolUse`       | Validate operations before execution |
| `PostToolUse`      | Track plan updates, format code      |
| `Stop`             | Remind about in-progress tracks      |
| `UserPromptSubmit` | Inject context into user prompts     |

## Agent Delegation Pattern

Instead of one massive prompt, delegate to specialists:

```markdown
# In /conductor-okf:new-track command

When generating the specification:

1. Use the Task tool with subagent_type='planner'
2. Prompt: "Generate a spec.md for: <description>"
3. The agent will handle the interactive Q&A

When the spec is approved:

1. Delegate plan generation to the same agent
2. Prompt: "Generate a plan.md based on the approved spec"
```

This keeps:

- Main command lightweight
- Specialized knowledge in agents
- Context windows clean
- Token usage efficient
