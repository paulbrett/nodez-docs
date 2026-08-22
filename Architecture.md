---
id: nodez-architecture
title: Architecture
type: architecture
status: active
created: 2026-08-19
updated: 2026-08-21
tags:
  - architecture
---

# Architecture

Nodez should be built in layers so storage and sync can evolve without rewriting the editor.

## Layers

1. App shell
2. Vault storage
3. Markdown editor
4. Indexer
5. Knowledge graph engine
6. Navigation and graph UI
7. GitHub sync

## Current prototype

```txt
Vite + React + TypeScript
  CodeMirror editor
  marked preview
  note index helpers
  localStorage persistence (browser) / vault files (Tauri)
  knowledge-graph engine (graph.ts): schema, provenance, filters, path, query
  graph engines (user-selectable):
    canvas2d GraphifyNetwork.tsx (default, Barnes-Hut)
    force3d ForceGraph3DNetwork.tsx (react-force-graph-3d, lazy-loaded)
  stdio MCP server (scripts/nodez-mcp.mjs)
  sync workflow panel
```

The Tauri desktop shell is now scaffolded (`src-tauri/`, Tauri v2): Rust
commands expose the vault filesystem and a change watcher, and the frontend
`vault.ts` adapter selects disk vs localStorage at runtime.

## Target desktop architecture

```txt
Tauri desktop shell
  React frontend
  Rust commands
    file system access
    file watcher
    git operations
    conflict detection
  local vault folder
  optional source folders
  graph database/output
```

## Unified Graph Architecture

Nodez should maintain two related indexes:

- **Vault index** for Markdown notes, frontmatter, wikilinks, tags, headings, backlinks, and unlinked mentions.
- **Project graph** for notes, files, code symbols, packages, components, decisions, features, artifacts, and their relationships.

The vault index powers fast note navigation. The project graph powers relationship queries, impact analysis, path finding, graph visualization, and agent context.

Edges should preserve provenance:

- `extracted` for relationships read directly from Markdown links, imports, calls, package manifests, YAML, or source references
- `inferred` for relationships resolved by static analysis or semantic extraction
- `manual` for explicit user-created relationships

## Important boundaries

- The editor should not know whether notes came from localStorage, disk, or GitHub.
- The sync layer should only operate on the vault folder.
- The indexer should rebuild from Markdown files and not require a database.
- App metadata should live in `.nodez/`, not inside note content.
- The graph layer must not invent implementation truth; it should point back to source files, notes, headings, or lines.

Related: [[Tauri Desktop Shell]], [[Vault Model]], [[GitHub Sync]], [[Unified Knowledge System]]
