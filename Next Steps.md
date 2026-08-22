---
id: diamante-next-steps
title: Next Steps
type: roadmap
status: active
created: 2026-08-20
updated: 2026-08-22
tags:
  - roadmap
  - planning
---

# Next Steps

Prioritized plan after the graph system, Tauri vault, repo indexing, command palette, and MCP write tools. Canonical agent/human onboarding plan: [[Agent and Human Setup]].

Ordered by dependency and leverage.

## Done foundation

### Phase 1 (Tauri vault)

- File explorer tree; create/rename/delete/move on disk; wikilink rewrite on rename
- Watcher reconciliation of external edits (skip active note content)
- Graph rebuild from notes follows vault automatically

### Phase 6a / 6b (repo source root + cheap extraction)

- Read-only source root; metadata index; git `branch|HEAD` re-index (not dirty thrash)
- Notes/Repo/Both origin; merged artifact at `.diamante/graph.json` (+ chunks)
- Markdown headings/artifacts, package `depends_on`, symbol extraction path, `.gitignore` respect

### Command palette + MCP writes (landed)

- In-app `Cmd+K` command palette
- MCP: graph query tools + `create_note` / `write_note` / `rename_note` / `delete_note` (soft trash)
- Dual Hermes servers: Dakila vault + Diamante docs vault

### Agent usefulness + human setup (P0–P4) — done 2026-08-21

Full checklist: [[Agent and Human Setup]].

1. **P0** — MCP note reads + note resources
2. **P1** — Graph freshness (`stale` / `rebuild_graph`)
3. **P2** — Setup wizard + one-click MCP export
4. **P3** — App-repo `AGENTS.md` agent contract
5. **P4** — `explain_edge`, `impact_of`, `list_communities`

### Release 0.2.0 — done 2026-08-21

- Version bump, Settings About, CHANGELOG, Windows MSI/NSIS, GitHub Release `v0.2.0` — [[Distribution Versioning and Updates]]

## Next: Editor UX — plain-text auto-format — **landed 2026-08-21 (first cut)**

When the user pastes or opens **plain / unformatted text** (not already structured Markdown):

1. **Detect** rough “unformatted” content — done (`looksUnformatted` in `src/frontmatter.ts`)
2. **Ask on paste** — done (confirm dialog in `MarkdownEditor` paste handler)
3. **Auto-format button** — done (banner in edit/preview when note looks plain)
4. **Preview frontmatter** — done (YAML stripped from preview body; Properties card)
5. Optional later: smarter format heuristics, session “always format”, selection-only format
6. **Preview polish (2026-08-22)** — GFM tables + syntax-highlighted code blocks (`src/markdownPreview.ts`, highlight.js)

App files: `src/frontmatter.ts`, `src/MarkdownEditor.tsx`, `src/App.tsx`, `src/styles.css`, `src/markdownPreview.ts`.

## Next: Lightweight code editor — **landed 2026-08-21 (MVP)**

Execute the plan in [[Code Editor Implementation]] (VS Code-like surface for code fences / optional code files):

- CodeMirror 6 stack — done (`src/components/code-editor/`)
- Format action (Prettier standalone) + lightweight lint diagnostics — done
- Theme-aware; lazy-loaded via `React.lazy` — done
- **Do not use Monaco for MVP** — done
- App wiring — topbar **Code** toggle (`SquareCode`); language select; Format; fence-aware round-trip for single-fence notes
- Still optional: deeper autocomplete, ESLint-in-browser, multi-file project awareness

## Then: richer agents + Dakila depth (P5)

- **P5** — Deeper code edges — **regex pass landed 2026-08-22** (cross-file calls, multi-line imports, methods); Dakila smoke OK — see [[Repo Indexing]]
- Optional later: tree-sitter WASM for full syntax fidelity

## Then: Phase 2 — GitHub sync (P6)

**v1 landed 2026-08-22** — real vault git in Rust + sync panel (see [[GitHub Sync]]):

- Detect repo; status (ahead/behind, changed files, conflicts)
- Pull (`--ff-only` then merge), stage, commit with generated/optional message, push
- Sync = pull → commit → push; **stops on conflicts** (no silent overwrite)
- Notes reload from disk after successful pull/sync

Still later: guided side-by-side conflict editor, clone/connect wizard, OAuth, auto graph rebuild on pull.

## Parallel: Graph UX and scale

Landed 2026-08-22 (this pass):

- **Node labels toggle** — floating FAB bottom-right; **hidden by default**; session-only state; 2D + 3D
- **Zoom to fit** — floating FAB stacked above labels control; both engines
- **3D SpriteText labels** with fixed light colors + hover tooltip contrast fix
- **Markdown preview** — proper GFM tables + highlight.js fenced code colors

Still open / next:

- ~~Layout cache in `.diamante/layout.json`~~ — **landed 2026-08-22** (2D cool-down + 3D engine-stop; Tauri + localStorage fallback)
- ~~Adjacency/path work in a worker~~ — **landed 2026-08-22** (`graphQueryWorker` holds full index; view/filter/path/draw off UI thread when ≥800 nodes)
- Optional community hulls, minimap
- Compact/mobile viewport review after workspace chrome changes

## Phase 3 knowledge features (remaining)

- Document outline, backlink context snippets, attachments, unresolved links
- Fast search ranking; keyboard shortcuts beyond the palette

## Cleanup and tech debt

- Delete `_to_delete/` when comfortable losing the superseded graph experiment
- Code-split remaining heavy paths; graph performance measurements on 1k demo
- Focused tests for note-tree collapse and new-folder reveal
- Harden CodeMirror paste path when adding auto-format (undo-friendly, selection-aware)

## Phase 5 polish and distribution (P7 remaining)

- Full icon set (`npm run tauri icon`); signed/notarized macOS; Windows/Linux installers polish
- **OTA** — Tauri updater, signed feed — [[Distribution Versioning and Updates]] (versioning + GitHub Releases baseline landed with 0.2.0)
- **Landing page** — public download/marketing site — [[Landing Page]]
- **No-Node MCP** path (bundled binary or Tauri-side server) for non-dev humans
- Export and mobile-responsive shell

## Explicit deprioritize

- In-app second full agent chat (prefer Hermes/CLI + Diamante MCP)
- Chat gateways inside Diamante
- MCP writes to indexed source roots

Related: [[Agent and Human Setup]], [[Code Editor Implementation]], [[Distribution Versioning and Updates]], [[Landing Page]], [[Product Roadmap]], [[Architecture]], [[GitHub Sync]], [[Tauri Desktop Shell]], [[Unified Knowledge System]], [[Graphify Tech Research]], [[Repo Indexing]], [[Command Palette and Agent Surface]], [[Frontend]]
