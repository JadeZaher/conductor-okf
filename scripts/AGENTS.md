# `scripts/` — build machinery

Not shipped. `build-plugin-zip.py` excludes `scripts/` and `dist/` from the
archive, so what lands on another machine is only the plugin itself.

## Why the runbook workflow is vendored rather than authored here

The `handoff` and `slice` skills and the `handoff`/`passoff` commands are
maintained at the **user level** (`~/.claude/skills/`, `~/.claude/commands/`) and
copied into this plugin at build time. That split is deliberate and it has a
cost, so it is worth stating the reasoning.

The skills are used constantly on the authoring machine, where they must work
*without* a plugin install and *without* a `conductor/` bundle — `handoff` inline
mode is the most-invoked path and creates nothing. Making the plugin the source
of truth would mean editing inside `plugins/marketplaces/local/`, which is
gitignored in the authoring `~/.claude` repo and shadowed by
`plugins/cache/local/<version>/` at load time. Two trees, neither tracked, both
loadable: that is how an edit silently fails to take effect.

So the canonical copy stays where it is edited and where git can see it, and the
build vendors it. The direction is one-way and the build is the only writer.
**Hand-editing `skills/handoff/`, `skills/slice/`, `commands/handoff.md`, or
`commands/passoff.md` inside this plugin works until the next build overwrites
it.**

## What the build rewrites, and why

A vendored file cannot keep the references that work at the user level, because
neither the path nor the bare command exists on a machine that only installed
this plugin:

| Source form | Vendored form | Why |
|---|---|---|
| `~/.claude/skills/…` | `${CLAUDE_PLUGIN_ROOT}/skills/…` | The user-level tree may not exist |
| `~/.claude/commands/…` | `${CLAUDE_PLUGIN_ROOT}/commands/…` | Same |
| `/handoff`, `/passoff`, `/slice` | `/conductor-okf:<name>` | Plugin commands are namespaced |

The command rewrite uses a lookbehind (`(?<![\w/.\-])`) so that path segments are
left alone — `skills/handoff/SKILL.md` and `../handoff/references/partitions.md`
must survive untouched, or every intra-skill link breaks. Relative links between
the two skills already resolve inside `skills/`, which is why `slice`'s
`../handoff/references/…` references need no rewrite at all.

## What is *not* rewritten

References to other plugins' commands (`/code-review`, `/security-review`,
`/simplify`, `/oh-my-claudecode:*`, `/codex:rescue`, …). Those are external by
design — `references/invocations.md` already tells the reader to verify a command
exists before naming it, and silently renaming them into this plugin's namespace
would invent commands that do not exist.

## Usage

```bash
python scripts/build-plugin-zip.py                    # vendor + validate + package
python scripts/build-plugin-zip.py --strict           # fail on validator warnings
python scripts/build-plugin-zip.py --skip-vendor      # package the tree as-is
python scripts/build-plugin-zip.py --claude-home DIR  # vendor from a different source
```

The archive is written to `dist/conductor-okf-<version>.zip`, versioned from
`.claude-plugin/plugin.json` — bump that first, and keep the enclosing
`marketplace.json` entry in agreement (`claude plugin tag` validates the pair).

The plugin root sits at the **archive root**: Claude Code extracts the zip and
treats the extracted directory as the plugin, so a wrapper folder would hide
`.claude-plugin/plugin.json` and the plugin would not load.
