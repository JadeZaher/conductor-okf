# Conductor for Claude Code

> Bringing Context-Driven Development to Claude Code

This documentation explores how [Conductor](https://github.com/gemini-cli-extensions/conductor), a Gemini CLI extension for Context-Driven Development, can be adapted and enhanced for Claude Code using its extensibility primitives: **Skills**, **Agents**, **Plugins**, **Commands**, and **Hooks**.

## Attribution

This project is inspired by and based on:

| | |
|---|---|
| **Original Project** | [Conductor](https://github.com/gemini-cli-extensions/conductor) |
| **Authors** | [gemini-cli-extensions](https://github.com/gemini-cli-extensions) |
| **License** | Apache License 2.0 |
| **Platform** | Gemini CLI |

## Overview

Conductor is a development methodology framework that transforms AI CLI tools into proactive project managers. It enforces a structured lifecycle for every task:

```
Context → Spec & Plan → Implement
```

The core philosophy: **treat context as a managed artifact alongside your code**, creating a single source of truth that provides persistent project awareness.

## Documentation Structure

| Document | Description |
|----------|-------------|
| [Conductor Analysis](./conductor-analysis.md) | Deep dive into the original Conductor architecture |
| [Claude Code Mapping](./claude-code-mapping.md) | How Conductor concepts map to Claude Code primitives |
| [Plugin Architecture](./plugin-architecture.md) | Proposed plugin structure and design |
| [Implementation Blueprint](./implementation-blueprint.md) | Step-by-step implementation guide |
| [Examples](./examples/) | Example command, agent, and skill files |

## Quick Comparison

| Aspect | Conductor (Gemini) | Conductor (Claude Code) |
|--------|-------------------|------------------------|
| Commands | TOML with inline prompts | Markdown with frontmatter |
| Context Loading | Manual per-command | Auto-discovered via Skills |
| Specialization | Single monolithic prompt | Dedicated Agents |
| User Interaction | Text-based A/B/C | Native AskUserQuestion UI |
| Event Handling | None | Hooks system |
| Distribution | Git clone | Plugin package |

## Key Benefits of Claude Code Implementation

1. **Skills provide auto-discovery** - Claude automatically follows TDD workflow when it detects a Conductor project
2. **Agents provide specialization** - Dedicated planning and implementation contexts
3. **Hooks provide automation** - Context loads automatically, progress tracked
4. **Plugins provide distribution** - Easy install, versioning, sharing

## Getting Started

See the [Implementation Blueprint](./implementation-blueprint.md) for how to build this plugin.

## References

- **Original Conductor**: https://github.com/gemini-cli-extensions/conductor
- **Claude Code Documentation**: https://docs.anthropic.com/en/docs/claude-code
- **Claude Code Plugins**: https://docs.anthropic.com/en/docs/claude-code/plugins
