---
name: code-styleguides
description: Language-specific code style guidelines. Use when writing TypeScript, Python, Go, JavaScript, or HTML/CSS code to ensure consistent, idiomatic, and maintainable code following best practices.
---

# Code Styleguides

Language-specific coding standards and best practices, plus the project's
cross-cutting standards. Load this skill before writing, reviewing or
refactoring code so findings cite the project's own rules.

## How It Works

Reads project-level styleguides from `conductor/code_styleguides/` (copied
during `/conductor-okf:setup` based on tech stack, then hardened per project).

## Reading order

1. `conductor/code_styleguides/engineering-principles.md` — the pillars every
   guide inherits (reusability, clean code, algorithmic excellence, validation
   & testing, portability). Read first, always.
2. Cross-cutting contracts when present:
   - `federation.md` — region manifest, per-layer source protocol, coverage
     declarations, readability-as-portability, soft size ceiling.
   - `layer-lanes.md` — the data-layer lane contract (lattice placement,
     nature, observed/forecast streams, provenance, availability).
   - `sql.md` — schema, migration and query standard.
3. The language guide for the file being touched (table below).

A reviewer cites the section (`federation.md §1`, `python.md "Readability and
region portability"`) so the author can locate the rule.

## Supported Languages

| Language | Extensions | Styleguide File |
|----------|------------|-----------------|
| TypeScript | `.ts`, `.tsx`, `.mts`, `.cts` | `conductor/code_styleguides/typescript.md` |
| Python | `.py`, `.pyi` | `conductor/code_styleguides/python.md` |
| SQL | `.sql`, Alembic `versions/*.py` | `conductor/code_styleguides/sql.md` |
| Go | `.go` | `conductor/code_styleguides/go.md` |
| JavaScript | `.js`, `.jsx`, `.mjs`, `.cjs` | `conductor/code_styleguides/javascript.md` |
| HTML/CSS | `.html`, `.css`, `.scss`, `.sass` | `conductor/code_styleguides/html-css.md` |

## When to Activate

Writing new code, reviewing code, refactoring, or setting up new files/modules.

## Universal Principles

| Aspect | Guideline |
|--------|-----------|
| **Naming** | Full words, intent-revealing; field-standard terms count as words; no cryptic abbreviations |
| **Structure** | Single responsibility; nesting ≤3; soft ceiling ~600-line modules / ~60-line functions (guidance, not lint) |
| **Documentation** | One-line "what" in code; the algorithm named at its implementation; the "why" in the directory's `AGENTS.md` with a pointer |
| **Errors** | Handle explicitly; fail closed and loud; never swallow; never substitute a plausible default for a missing governed input |
| **Boundaries** | Validate every external value once at ingress and give it a named type |
| **Portability** | No footprint literal (coordinate, state code, region name) outside the region manifest; sources bind behind a per-layer protocol and declare coverage |
| **Testing** | Behaviour and its failure/partial paths; one full sweep at the end of a multi-fix pass |

## Guide-consistency rule

A change that makes any rule in a guide false fixes the rule in the same
change and names the ruling or measurement that supersedes it. A checklist
item that has outlived its decision fails review for the wrong reason.

## Setup

If styleguides missing:
1. Run `/conductor-okf:setup` to initialize
2. Or copy from `templates/code-styleguides/` to `conductor/code_styleguides/`

Check: `ls conductor/code_styleguides/`

## Integration

Works with: **conductor-context** (project overrides), **tdd-workflow** (language test patterns)
