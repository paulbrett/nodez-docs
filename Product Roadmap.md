---
id: diamante-product-roadmap
title: Product Roadmap
type: roadmap
status: active
created: 2026-08-19
updated: 2026-08-20
tags:
  - roadmap
---

# Product Roadmap

## Status (2026-08-20)

- Phase 0 (Prototype): done.
- Phase 1 (Real vaults): done — Tauri v2 shell builds and runs (verified end to end with a real Rust toolchain); open-folder, read/write/create, file explorer tree, rename/delete with cross-note `[[wikilink]]` rewriting, and watcher-based reconciliation of external edits are all in place.
- Phase 3 (Knowledge features): partial — richer graph view, frontmatter parsing, themes, settings, empty states, and Obsidian-like note tree controls landed; command palette, outline, backlink snippets, and attachments pending.
- Phase 4 (Unified graph system): largely done — schema, provenance, local/global modes, filters, path finder, query/explain, and an MCP server exist; rebuild-after-sync pending (waits on Phase 2).
- Phase 2 (GitHub sync) and Phase 5 (Polish): not started.
- Phase 6 (External repo/project indexing): Phase 6a and 6b done — source-root picker, recursive index, git-status re-indexing, graph merge, Notes/Repo/Both toggle, visible Open Vault/Open Repo controls, Markdown heading extraction, Markdown repo/vault references, package dependency extraction, and `.diamante/graph.json` artifact metadata. Next: command palette/agent surface, then tree-sitter code edges. First target is the Dakila repo (`OverlandLightingControllerV1`) at `/Users/paulbrettorozco/Sites/overland`.

## Phase 0 - Prototype

- React app scaffold
- CodeMirror editor
- Markdown preview
- local note persistence
- wikilinks
- backlinks
- tags
- graph preview
- GitHub sync workflow panel

## Phase 1 - Real vaults

- Tauri app shell
- open folder as vault
- read/write `.md` files
- file explorer
- filesystem watcher
- real note create, rename, delete

## Phase 2 - GitHub sync

- detect git repo
- connect remote
- pull
- commit
- push
- conflict UI
- sync history

## Phase 3 - Knowledge features

- command palette
- better graph view
- backlinks with context snippets
- outline
- properties/frontmatter
- attachments

## Phase 4 - Unified graph system

- graph node and edge schema
- local graph and global graph modes
- edge provenance
- graph search and filters
- path finder between notes, files, symbols, and decisions
- query/explain panel
- graph rebuild after git sync
- agent-first graph query instructions

## Phase 5 - Polish

- themes
- keyboard shortcuts
- export
- mobile-responsive shell
- package installers

## Phase 6 - External repo/project indexing

- open a second, read-only "source root" alongside the notes vault (a plain project/code folder, not a vault of notes) — done
- persist the merged graph artifact in the opened vault at `.diamante/graph.json` — done
- extract file/folder nodes, package-manifest `depends_on` edges, and Markdown docs from the source root the same way the vault already does — done
- merge the source-root graph with the vault graph into one unified graph (same node/edge schema already in [[Unified Knowledge System]]) — done
- tree-sitter extraction for `imports`/`calls`/`defines` edges
- MCP server exposes both the vault and the source root as one workspace
- first concrete target: the Dakila repo (`OverlandLightingControllerV1`) at `/Users/paulbrettorozco/Sites/overland` — see [[Repo Indexing]]
