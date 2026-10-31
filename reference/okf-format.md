# OKF Convention for Conductor

Conductor's `conductor/` directory is an **Open Knowledge Format (OKF v0.1)** bundle:
a directory of markdown "concepts", each carrying YAML frontmatter. This file is the
single source of truth for the convention — other plugin files point here instead of
repeating it.

> OKF is a vendor-neutral standard (Google, v0.1) for representing context/knowledge
> as a directory of markdown files with YAML frontmatter. Only `type` is required;
> everything else is producer-defined. See
> https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing

## Frontmatter schema

Every conductor markdown file begins with a frontmatter block:

```yaml
---
type: <required — the concept type, see taxonomy below>
title: <optional human title>
description: <optional one-line summary>
tags: <optional [list]>
timestamp: <optional ISO 8601, last modified>
resource: <optional URL or repo-relative path to the underlying resource>
---
```

`type` is the **only** required field. Add others when they add signal; don't pad.
The body below the frontmatter is normal markdown — keep conductor's existing section
structure unchanged.

## Type taxonomy (conductor concepts)

| File | `type` |
|------|--------|
| `product.md` | `Product Overview` |
| `product-guidelines.md` | `Product Guidelines` |
| `tech-stack.md` | `Tech Stack` |
| `workflow.md` | `Workflow` |
| `tracks.md` | `Track Index` |
| `code_styleguides/*.md` | `Code Styleguide` |
| `tracks/<id>/spec.md` | `Track Spec` |
| `tracks/<id>/plan.md` | `Implementation Plan` |

`metadata.json` stays as-is — it remains the machine-readable source of truth for the
plugin's hooks. OKF frontmatter on `spec.md`/`plan.md` complements it (mirror the track
id and status into `tags`/`resource`; don't duplicate the whole file).

## Cross-linking

Link related concepts with normal markdown links (this is how OKF forms a graph):
`plan.md` links to its `./spec.md`; `spec.md` links to `../../tech-stack.md` and
`../../product.md` where relevant. `tracks.md` is the bundle's `index.md` analog.

## Example: `tracks/<id>/spec.md`

```yaml
---
type: Track Spec
title: Add OAuth login
description: Spec for the OAuth login feature track.
tags: [feature, add_oauth_login_20260627, pending]
timestamp: 2026-06-27T00:00:00Z
resource: ./metadata.json
---

# Overview
...
```

## Lazy migration (the "update existing tracks" rule)

When working in a `conductor/` project, if a context or track markdown file is
**missing** OKF frontmatter, add it:

1. Insert a frontmatter block at the very top using the type from the taxonomy.
2. Do **not** rewrite or reorder the existing body — frontmatter is additive.
3. Derive `title`/`description` from the file's existing heading; set `tags` from the
   track type/status where known; stamp `timestamp` with the current date only if you
   can determine it (otherwise omit).

This is non-destructive and safe: hooks parse `plan.md` by `## Phase` / `- [ ]` line
markers, which frontmatter never matches.
