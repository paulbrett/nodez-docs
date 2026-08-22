---
id: diamante-product-roadmap
title: Product Roadmap
type: roadmap
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - roadmap
---

# Product Roadmap

## Status (2026-08-22)

- Phase 0 (Prototype): done.
- Phase 1 (Real vaults): done — Tauri v2 shell; open-folder; read/write/create; explorer; rename/delete with wikilink rewrite; external-edit watcher.
- Phase 3 (Knowledge features): partial — outline + matching backlink/outgoing pills landed; flush full-height inspector; attachments / search ranking pending — [[Next Steps]]. Plain-text auto-format + code editor MVP done — [[Code Editor Implementation]].
- Phase 4 (Unified graph system): largely done — schema, provenance, local/global, filters, path finder, query/explain UI, MCP + freshness + dual Hermes; notes reload after pull (graph artifact debounce) — [[Agent and Human Setup]].
- Phase 2 (GitHub sync): **v1 landed** — vault status/pull/commit/push/sync + conflict list + sync panel; control lives in notes footer — [[GitHub Sync]].
- Phase 5 (Polish): partial — themes, Windows MSI/NSIS; **landing + OTA skeleton landed** (in-repo `landing/`, updater plugins, Pages/release workflows); icons/Authenticode, no-Node MCP, broader shortcuts pending.
- Phase 6 (External repo/project indexing): 6a/6b + P5 deeper extraction (cross-file calls) landed — [[Repo Indexing]].
- **Active track:** finish P7 (Pages enable, signed `v0.3.0` feed) or remaining Phase 3 (attachments, search) — [[Next Steps]].

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

- detect git repo — done (v1)
- pull / commit / push / sync — done (v1); host git auth
- conflict list (stop, no silent overwrite) — done (v1)
- sync panel + notes-footer control — done
- connect/clone wizard, OAuth, guided conflict editor — later
- graph artifact refresh after pull — via note reload + debounced save

## Phase 3 - Knowledge features

- command palette — done
- better graph view — largely done (2D/3D engines, filters, path, explain)
- backlinks (title pills, same chrome as outgoing) — done
- outline (full list, jump/scroll) — done
- unresolved outgoing markers — done
- properties/frontmatter — partial
- attachments — pending
- first-run wizard / MCP export — done (P2)
- plain-text / paste auto-format — done first cut — [[Next Steps]]
- lightweight code editor mode — done MVP — [[Code Editor Implementation]]

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
- package installers — MSI/NSIS
- app versioning (single source of truth, About UI, semver tags) — done baseline — [[Distribution Versioning and Updates]]
- OTA updates via Tauri updater + signed feed — **skeleton 2026-08-22** (plugins, Settings check, Pages endpoint); first signed platform payload pending — [[Distribution Versioning and Updates]]
- public landing page (download + positioning) — **shipped in-repo plain HTML/CSS** `landing/` — [[Landing Page]]
- no-Node / bundled MCP for non-dev setup — still P7 remaining

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
| P5 | Deeper code extraction (Dakila) — **regex pass landed**; tree-sitter optional |
| P6 | Real git sync + rebuild-after-pull — **v1 landed** (panel + footer control) |
| P7 | Distribution — **landing + OTA skeleton landed**; remaining: signed tag feed, Authenticode, no-Node MCP, icons |
