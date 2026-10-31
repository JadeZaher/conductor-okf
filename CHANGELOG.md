# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0] - 2026-08-27

Fork lineage note: entries at `1.2.0` and below are upstream claude-conductor's.
This fork detached at upstream 1.2.0 and versioned itself `1.1.0`; `1.3.0` is the
first fork release numbered above the inherited history, so the two never collide
again.

### Added
- **Runbook workflow, vendored into the plugin.** The `handoff` and `slice` skills
  and the `/conductor-okf:handoff` + `/conductor-okf:passoff` commands now ship with
  the plugin, so one install carries the whole context-driven loop instead of
  requiring user-level skills alongside it.
  - `skills/handoff/` — owns `conductor/RUNBOOK.md` end to end: inline packets,
    durable writes, rolling checkpoints, and `resume`.
  - `skills/handoff/references/` — the target rule (`runbook.md`), the one-round
    grilling gate (`grilling.md`), rolling checkpoints and phase-boundary review
    verdicts (`rolling.md`), the invocation weight ladder (`invocations.md`), and the
    prebaked-partition schema (`partitions.md`), shared by both skills.
  - `skills/slice/` — the parallel sibling: conflict-free file partitions, wave plans
    derived from `depends_on`, and a pre-launch grilling gate.
- **Zip distribution.** `scripts/build-plugin-zip.py` vendors the workflow from its
  canonical user-level source, rewrites user-level paths to `${CLAUDE_PLUGIN_ROOT}`
  and bare slash commands to namespaced ones, runs `claude plugin validate`, and
  writes `dist/conductor-okf-<version>.zip` with the plugin root at the archive root.
- README: install-from-zip instructions (`--plugin-dir`, `--plugin-url`, and a
  `{"source": "url"}` marketplace entry) and a Runbook Workflow section.

### Notes
- The partition schema the skills document is the same `partitions` object
  `/conductor-okf:new-track` writes into `conductor/tracks/<id>/metadata.json`, so the
  boundary reasoning done at planning time is what `/conductor-okf:slice` consumes.
- No runbook *command* is added: `skills/handoff` is the sole writer of
  `conductor/RUNBOOK.md`, by design.

## [1.2.0] - 2025-12-29

### Changed
- Session resumption now relies solely on plan.md status markers instead of automatic context threshold tracking

### Removed
- Automatic context threshold tracking and handoff mechanism
- context-threshold.js hook and related Task hook configuration
- Handoff sections from implement command documentation

### Documentation
- Added session resumption documentation to README.md
- Simplified workflow documentation

## [1.1.1] - 2024-12-27

### Added
- Agents and commands now inherit model from parent context when not explicitly set

### Fixed
- Prevent CPU spikes from unbounded parallel file operations in hooks
- Optimize stdin reading in all hook scripts (array collection vs string concatenation)
- Add orphan worktree detection warning on session start
- Document resource limits for concurrent background agents

## [1.0.1] - 2024-12-24

### Fixed
- Agent type references in commands now use fully-qualified names (conductor:planner, conductor:implementer, conductor:reviewer)

## [1.0.0] - 2024-12-24

### Added
- Skills preloading, background agents, and worktree isolation
- Plugin marketplace support with marketplace.json
- Single-phase execution mode with interactive selection
- Context threshold auto-handoff feature
- Code styleguides moved to templates system
- Phase 5 templates and documentation
- Phase 4 hooks automation
- Phase 3 skills integration
- Graceful file existence checks before reading
- Context threshold notification when reached

### Performance
- Parallel tool call instructions across all commands and agents
- Optimized plugin for reduced token usage and faster execution

### Changed
- Updated command references to use kebab-case
- Renamed agents to remove conductor- prefix
- Set explicit models for agents and commands
- Updated model strings throughout

### Fixed
- Source path now uses ./ prefix in marketplace.json
- Improved error handling with graceful file checks

### Documentation
- Updated README title to Claude Conductor
- Enhanced installation instructions with GitHub marketplace
- Added specific Bash commands for pre-flight checks

## [0.1.0] - Initial Release

### Added
- Initial Context-Driven Development framework
- Basic planner, implementer, and reviewer agents
- Core command structure
