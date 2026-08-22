---
id: diamante-product-roadmap
title: Product Roadmap
type: roadmap
status: active
created: 2026-08-19
updated: 2026-08-21
tags:
  - roadmap
---

# Product Roadmap

## Status (2026-08-21)

- Phase 0 (Prototype): done.
- Phase 1 (Real vaults): done — Tauri v2 shell; open-folder; read/write/create; explorer; rename/delete with wikilink rewrite; external-edit watcher.
- Phase 3 (Knowledge features): partial — richer graph, frontmatter, themes, settings, note tree, command palette; outline, backlink snippets, attachments pending; **plain-text auto-format** and **lightweight code editor** planned — [[Next Steps]], [[Code Editor Implementation]].
- Phase 4 (Unified graph system): largely done — schema, provenance, local/global, filters, path finder, query/explain UI, MCP server + write/read tools + freshness (`rebuild_graph` / `stale`) + dual Hermes wiring; rebuild-after-sync pending (waits on Phase 2) — see [[Agent and Human Setup]].
- Phase 2 (GitHub sync): not started (preview UI only).
- Phase 5 (Polish): partial — themes, Windows MSI/NSIS; icons/signing, no-Node MCP, broader shortcuts pending.
- Phase 6 (External repo/project indexing): 6a/6b done — source root, commit-driven re-index, merge artifact, extraction for docs/manifests/symbols, gitignore toggle, graph engines 2D/3D. Next extraction: deeper code edges; first target Dakila — [[Repo Indexing]].
- **Active track:** Agent usefulness + human setup — **P0–P4 done**; next **P5** Dakila extraction (or P6/P7 by choice) — [[Agent and Human Setup]], [[Next Steps]].

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
- graph rebuild after pull (agent artifact freshness)

## Phase 3 - Knowledge features

- command palette — done
- better graph view — largely done (2D/3D engines, filters, path, explain)
- backlinks with context snippets
- outline
- properties/frontmatter — partial
- attachments
- first-run wizard / MCP export — done (P2)
- plain-text / paste auto-format (ask first + Auto-format button) — [[Next Steps]]
- lightweight code editor mode — [[Code Editor Implementation]]

## Phase 4 - Unified graph system

- graph node and edge schema
- local graph and global graph modes
- edge provenance
- graph search and filters
- path finder between notes, files, symbols, and decisions
- query/explain panel
- graph rebuild after git sync
- agent-first graph query instructions
- MCP note read tools + freshness contract — [[Agent and Human Setup]] P0–P1
- higher-order tools: impact, explain edge, communities — P4

## Phase 5 - Polish

- themes
- keyboard shortcuts
- export
- mobile-responsive shell
- package installers
- no-Node / bundled MCP for non-dev setup — P7
- app versioning (single source of truth, About UI, semver tags) — [[Distribution Versioning and Updates]]
- OTA updates via Tauri updater + signed feed — [[Distribution Versioning and Updates]]
- public landing page (download + positioning) — [[Landing Page]]

## Phase 6 - External repo/project indexing

- open a second, read-only "source root" alongside the notes vault — done
- persist the merged graph artifact in the opened vault at `.diamante/graph.json` — done
- extract file/folder nodes, package-manifest `depends_on` edges, and Markdown docs — done
- merge source-root graph with vault graph — done
- tree-sitter (or equivalent) extraction for stronger `imports`/`calls`/`defines` edges
- MCP exposes vault + source root as one **read** workspace (writes remain vault-only)
- first concrete target: Dakila `OverlandLightingControllerV1` — [[Repo Indexing]]

## Phase 7 - Agent and human setup (named track)

Not a replacement for Phases 2–6; a cross-cutting delivery order documented in [[Agent and Human Setup]]:

| Priority | Outcome |
| --- | --- |
| P0 | MCP `list` / `search` / `read` notes + resources — **done 2026-08-21** |
| P1 | Graph artifact freshness after agent writes — **done 2026-08-21** |
| P2 | First-run wizard + one-click MCP config export — **done 2026-08-21** |
| P3 | App `AGENTS.md` agent contract — **done 2026-08-21** |
| P4 | Impact / explain / communities tools — **done 2026-08-21** |
| P5 | Deeper code extraction (Dakila) |
| P6 | Real git sync + rebuild-after-pull |
| P7 | Distribution: versioning, OTA, landing page, no-Node MCP, icons, signed builds |
