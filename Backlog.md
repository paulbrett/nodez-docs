---
id: diamante-backlog
title: Backlog
type: backlog
status: active
created: 2026-08-19
updated: 2026-08-20
tags:
  - backlog
---

# Backlog

## Editor

- Add slash command menu
- Add formatting toolbar
- Add image and file embeds
- Add heading outline
- Add keyboard shortcuts

## Vault

- Open local folder — done
- Create note in folder — done
- Rename note and update links — done
- Delete note with confirmation — done
- Move notes between folders — done
- Create empty folders — done
- First vault load collapses folders by default — done

## Sync

- Show git status
- Pull from remote
- Commit local changes
- Push to GitHub
- Conflict resolution screen

## Index

- Better tag parsing
- Unresolved links
- Backlink snippets
- Fast search ranking
- Graph filters and zoom controls
- Graph search and connection highlighting
- Expand node and edge schema beyond notes/tags
- Add inferred and manual edge provenance
- Local graph depth control
- Query, path, and explain graph tools
- Source references from graph edges back to notes/files/lines
- Lazy-load the full graph renderer so the main app bundle stays small
- Add graph performance measurements using the 1,000-node dummy vault fixture

## Dakila Workflow

- Open docs vault and source repo as one workspace
- Graph query before broad file search
- Source-of-truth conflict banner
- Post-change reminder to update docs and graph
- GitHub sync flow for code/docs/graph outputs

## Repo Indexing (see [[Repo Indexing]])

- Read-only source-root command (list files/folders of an arbitrary project folder, no note semantics) — done
- Register a source root (e.g. the Dakila repo) alongside the vault path — done
- `file`/`folder` graph nodes for the source root, containment edges only — done
- Save merged vault+repo graph artifact inside the vault at `.diamante/graph.json` — done
- Restore visible Open Vault/Open Repo controls in the simplified UI — done
- Parse the source root's Markdown docs and `package.json` manifests into `documents`/`references`/`depends_on` edges — done
- Preserve extracted repo metadata in the Graphify-compatible `.diamante/graph.json` artifact — done
- Tree-sitter extraction for C++ and TypeScript/TSX (the languages the Dakila repo actually uses)
- Merge the source-root graph into the same unified graph as the vault
- Extend the MCP server's tool surface to cover the source root
- Filesystem watcher + rebuild for the source root, matching the vault's watcher

## Design

- Expand and refine color themes — done for Graphite, Paper, and Contrast; more can still be added later
- Empty states — partial: empty editor/vault state is in place; compact/mobile still needs hands-on review
- Settings screen — done
- Command palette
- Light and dark themes
