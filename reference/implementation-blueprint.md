# Implementation Blueprint

> Step-by-step guide to building Conductor for Claude Code

## Overview

This blueprint outlines the phased approach to implementing Conductor as a Claude Code plugin.

## Phase 1: Core Plugin Structure

### Goals

- Create plugin manifest
- Set up directory structure
- Implement basic command scaffolding

### Tasks

#### 1.1 Create Plugin Skeleton

```bash
mkdir -p conductor-claude/{.claude-plugin,commands,agents,skills,hooks,templates}
```

#### 1.2 Create Plugin Manifest

**`.claude-plugin/plugin.json`**

```json
{
  "name": "conductor",
  "version": "0.1.0",
  "description": "Context-Driven Development framework",
  "commands": ["./commands/"],
  "agents": "./agents/",
  "skills": "./skills/",
  "hooks": "./hooks/hooks.json"
}
```

#### 1.3 Port Commands (Basic)

Start with simplified versions of each command:

1. **setup.md** - Project initialization
2. **new-track.md** - Track creation
3. **implement.md** - Task execution
4. **status.md** - Progress display
5. **revert.md** - Smart rollback

#### 1.4 Validation

- Install plugin locally
- Test each command runs without errors
- Verify namespace (`/conductor-okf:*`) works

---

## Phase 2: Agent Specialization

### Goals

- Create dedicated agents for planning, implementation, review
- Wire commands to delegate to agents
- Test agent handoffs

### Tasks

#### 2.1 Create Planner Agent

**`agents/planner.md`**

Focus on:

- Requirements extraction
- Spec.md generation
- Plan.md generation with TDD structure

#### 2.2 Create Implementer Agent

**`agents/implementer.md`**

Focus on:

- TDD cycle execution
- Plan progress tracking
- Quality gate enforcement

#### 2.3 Create Reviewer Agent

**`agents/reviewer.md`**

Focus on:

- Test coverage verification
- Manual verification plans
- Checkpoint creation

#### 2.4 Update Commands to Delegate

Modify commands to use Task tool with agents:

```markdown
## Specification Generation

Delegate to the planner agent:

- Use Task tool with subagent_type='planner'
- Provide the track description as context
- Wait for spec.md content
```

#### 2.5 Validation

- Test planning flow creates valid specs
- Test implementation follows TDD
- Test review creates checkpoints

---

## Phase 3: Skills Integration

### Goals

- Create auto-discovered capabilities
- Implement context awareness
- Port code styleguides

### Tasks

#### 3.1 Create Context Awareness Skill

**`skills/context-awareness/SKILL.md`**

Auto-loads conductor context when directory detected.

#### 3.2 Create TDD Workflow Skill

**`skills/tdd-workflow/SKILL.md`**

Provides TDD guidance during implementation.

#### 3.3 Port Code Styleguides

**`skills/code-styleguides/`**

- SKILL.md (auto-activation skill, reads from project)

**`templates/code-styleguides/`**

- typescript.md (template copied to project during setup)
- python.md
- go.md
- javascript.md
- html-css.md

#### 3.4 Validation

- Verify skills auto-discover in conductor projects
- Test TDD skill activates during implementation
- Test styleguide skill with different languages

---

## Phase 4: Hooks & Automation

### Goals

- Implement event-driven behaviors
- Auto-load context on session start
- Remind about in-progress work

### Tasks

#### 4.1 Create Hooks Configuration

**`hooks/hooks.json`**

Implement:

- SessionStart: Load conductor context
- PostToolUse: Track plan modifications
- Stop: Remind about in-progress tracks

#### 4.2 Test Event Triggers

- Start new session in conductor project
- Verify context message appears
- Make changes to plan.md
- Verify tracking works
- End session
- Verify reminder appears

---

## Phase 5: Templates & Polish

### Goals

- Create default templates
- Add comprehensive error handling
- Write documentation

### Tasks

#### 5.1 Create Templates

Port from original Conductor:

- workflow.md
- product.md template
- product-guidelines.md template
- Code styleguide templates

#### 5.2 Error Handling

Add to all commands:

- Pre-flight checks for conductor directory
- Validation of required files
- Graceful failure with helpful messages

#### 5.3 Documentation

- README.md with installation instructions
- Command reference
- Configuration options
- Troubleshooting guide

---

## Phase 6: Testing & Release

### Goals

- End-to-end testing
- Performance optimization
- Release preparation

### Tasks

#### 6.1 End-to-End Testing

Full workflow test:

1. Fresh project → /conductor-okf:setup
2. Create track → /conductor-okf:new-track
3. Implement → /conductor-okf:implement
4. Check status → /conductor-okf:status
5. Revert if needed → /conductor-okf:revert

#### 6.2 Edge Cases

Test:

- Brownfield project detection
- Resume interrupted setup
- Multiple tracks
- Phase completion checkpoints
- Git revert scenarios

#### 6.3 Performance

- Minimize unnecessary file reads
- Optimize hook execution
- Test with large projects

#### 6.4 Release

- Tag version 1.0.0
- Publish to GitHub
- Add to Claude Code plugin registry (if available)

---

## Key Implementation Details

### State Management

Use JSON files for state, loaded via hooks:

```json
// conductor/setup_state.json
{
  "last_successful_step": "2.3_tech_stack",
  "project_type": "brownfield",
  "created_at": "2024-12-22T10:00:00Z"
}
```

### Interactive Patterns

Use structured approach for user interaction:

```markdown
Present options to user:

1. Read current state
2. Generate options based on context
3. Present as clear choices
4. Process response
5. Update state
6. Continue or loop
```

### Git Integration

Leverage Bash tool for git operations:

```markdown
## Commit Changes

1. Stage files: `git add <files>`
2. Create commit: `git commit -m "<type>(<scope>): <description>"`
3. Add git note: `git notes add -m "<summary>" <sha>`
4. Update plan with SHA
```

### Error Recovery

Every command should:

1. Check preconditions first
2. Fail fast with clear messages
3. Preserve state for resume
4. Suggest next steps

---

## Development Workflow

### Local Testing

```bash
# Install plugin locally
cd ~/.claude/plugins/
ln -s /path/to/conductor-claude conductor

# Or add to settings.json
{
  "plugins": ["/path/to/conductor-claude"]
}
```

### Iterative Development

1. Modify command/agent/skill
2. Restart Claude Code session
3. Test changes
4. Repeat

### Debugging

Use verbose mode to see hook output:

```bash
claude --verbose
```

Check hook execution in debug logs:

```
~/.claude/debug/
```

---

## Success Criteria

### Functional

- [ ] All 5 commands work correctly
- [ ] Agents delegate appropriately
- [ ] Skills auto-discover
- [ ] Hooks fire on events
- [ ] State persists across sessions

### Quality

- [ ] Clear error messages
- [ ] Graceful failure handling
- [ ] Consistent UI patterns
- [ ] Good documentation

### Performance

- [ ] Commands complete in reasonable time
- [ ] Minimal token overhead
- [ ] Efficient file operations
