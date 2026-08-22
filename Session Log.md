---
id: diamante-session-log
title: Session Log
type: session-log
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - session-log
  - repo-indexing
---

# Session Log

## 2026-08-22 (landing repo split)

Split public site out of private app repo:

- New **public** repo `paulbrett/diamante-landing` at `C:\Sites\diamante-landing`
- Pages live: `https://paulbrett.github.io/diamante-landing/`
- Removed in-app `landing/` + app Pages workflow; app release pushes feed via `LANDING_DEPLOY_TOKEN`
- Updater endpoint + Settings About link updated

## 2026-08-22 (P7 landing + OTA skeleton)

Shipped distribution surface in app repo (`paulbrett/diamante` `26ddb9d`):

- **Landing** — single-page plain HTML/CSS under `landing/` (in main repo, not separate site)
- **OTA** — manifest `landing/updates/latest.json`, bundles dir, Tauri updater + process plugins, Settings → Check for updates
- Pages + Windows release workflows; public key in `tauri.conf.json`; private key CI secret only
- Docs: [[Landing Page]], [[Distribution Versioning and Updates]], [[Next Steps]], [[Product Roadmap]], [[Agent and Human Setup]]

App already on main. Next human steps: enable Pages, set `TAURI_SIGNING_PRIVATE_KEY`, tag `v0.3.0`.

## 2026-08-22 (vault agent contract)

Shipped opt-in vault **AGENTS.md** for end-user projects: managed markers, foreign-root → `.diamante/AGENTS.md`, merge action, Setup **Agents** step, Settings + palette. Pure logic `src/agentContract.ts` + tests. Docs: [[Agent Skills and Surfaces]].

## 2026-08-22 (agent surfaces)

Added canonical map [[Agent Skills and Surfaces]]: Layer 1 MCP + app `AGENTS.md`, Layer 2 host skills (Hermes), Layer 3 vault knowledge; placement tree; linked from [[Home]], [[Next Steps]], [[Agent and Human Setup]], [[Command Palette and Agent Surface]].

## 2026-08-22

Workspace chrome + Phase 3 inspector polish (app `C:\Sites\diamante`):

- **GitHub sync v1** already on main; moved control to notes-tree footer beside note count (status-bar height/type).
- Notes list sits flush on the count bar; counter padding tightened to shell.
- **Inspector** is a full-height workspace grid column flush top/right/bottom (not nested in main grid gutters).
- **Outline** lists all headings (no internal scroll); click jumps edit / scrolls preview.
- **Backlinks** and **Outgoing** share pill chrome (titles only; unresolved outgoing `· ?`).
- Docs: [[Next Steps]], [[Product Roadmap]], [[GitHub Sync]], [[Frontend]] chrome note.

## 2026-08-19

Created the first Diamante Notes prototype.

Implemented:

- Vite React TypeScript app
- CodeMirror Markdown editor
- Markdown preview
- localStorage note persistence
- seed notes
- wikilink extraction
- backlink panel
- outgoing link panel
- tag extraction
- GitHub sync workflow panel
- mini graph visualization

Verified:

- `npm install`
- `npm run build`
- local dev server at `http://127.0.0.1:5173/`

Next:

- convert browser prototype to real vault storage
- add Tauri shell
- connect actual git operations

## 2026-08-19 Theme Pass

Updated the app UI direction:

- dark theme is now the default
- added switchable color themes
- added theme persistence
- updated CodeMirror styling to follow the active theme
- documented the feature in the app README

## 2026-08-19 Markdown and YAML Check

Added documentation quality checks:

- YAML frontmatter on every vault Markdown note
- Markdown lint script
- frontmatter validation script
- `npm run check` command in the app repo

## 2026-08-19 Graph Preview Modal

Updated the graph experience:

- moved graph preview above the Backlinks panel
- removed the graph from the sync sidebar
- added a full graph modal with note connection lines
- made graph nodes selectable from the modal

## 2026-08-19 Unified Obsidian and Graphify Direction

Researched and documented the product direction:

- Obsidian-inspired local vault, backlinks, YAML properties, local/global graph, and graph filters
- Graphify-inspired relationship graph, edge provenance, communities, path/query/explain workflow, and source-backed graph outputs
- Dakila-style workflow where GitHub/source is implementation truth, vault notes are intent, and the graph layer explains relationships
- added [[Unified Knowledge System]] as the canonical note for this direction

## 2026-08-20 Claude Graph Research Artifact

Recovered the Claude-created [[Graphify Tech Research]] artifact from the Diamante docs vault.

Integrated it into the vault index and docs checks:

- linked it from [[Home]]
- linked it from [[README]]
- allowed `research` as a docs frontmatter type

## 2026-08-20 Graph Experiment Integration

Integrated the graph research experiment into the app prototype:

- added a typed graph schema for `note` and `tag` nodes
- added `links_to` and `tagged_with` edges
- added edge provenance, confidence, and source path metadata
- replaced the ad hoc modal graph with schema-backed local/global graph modes
- added graph search, node type filters, stats, and an explain panel
- kept the graph grounded in extracted vault data only

## 2026-08-20 Graphify MCP and Canvas Graph

Implemented the verified Graphify direction in the real app at `/Users/paulbrettorozco/Sites/Diamante`:

- expanded the graph schema to the full note/file/symbol/package/decision/feature/component/tag/artifact model
- added Graphify JSON import/export helpers and relationship query/path/hub utilities
- replaced the full graph modal with a `react-force-graph-2d` canvas renderer
- added provenance, relationship type, local depth, hub, and shortest-path controls
- added a stdio MCP server that reads the app source folder and the Obsidian vault at `/Users/paulbrettorozco/Documents/Projects/Diamante`
- verified `npm run lint`, `npm run build`, and MCP `graph_stats` / `shortest_path` smoke calls

## 2026-08-20 Graphify Renderer Alignment

Updated the full graph modal to use the same visualization family as Graphify's
open-source HTML exporter:

- replaced `react-force-graph-2d` with `vis-network@9.1.6`
- added `src/GraphifyNetwork.tsx` as the isolated Graphify-style renderer
- matched Graphify's ForceAtlas2-style physics constants, stabilization, arrows,
  dot nodes, degree-based sizing, hover tooltips, and confidence-styled edges
- added a Graphify-style community legend with show/hide filtering
- preserved Diamante's local/global modes, node/edge/provenance filters,
  shortest-path highlighting, hub list, and source-backed Explain panel
- visually verified the graph modal in the browser at `http://127.0.0.1:5173/`
- verified `npm run build`

## 2026-08-20 Dummy Graph Stress Data

Added a deterministic dummy vault fixture for graph scale testing:

- added `generateDummyNotes()` in the app seed module
- added a sidebar `Load 1k demo` action that opens the global graph view
- fixture generates 940 notes and 60 tag nodes, producing exactly 1,000 graph nodes
- generated graph currently produces 5,640 extracted edges from wikilinks and tags
- verified through the real browser UI with no console warnings
- verified `npm run build`

## 2026-08-20 Canvas Renderer, Full-Screen Graph, and Tauri Shell

Replaced the graph renderer, enlarged the graph modal, and scaffolded the desktop shell.

Graph renderer:

- replaced `vis-network` with an in-house, dependency-free canvas force engine in `src/GraphifyNetwork.tsx` (identical props, true drop-in, so `App.tsx` was untouched)
- added Barnes-Hut quadtree repulsion so the 1,000-node demo stays smooth
- kept community color, degree-based sizing, provenance-styled edges (solid extracted, dashed inferred/manual), node and edge selection, neighbor dimming, shortest-path highlighting, and theme-following via CSS variables
- fixed a force-explosion (NaN) bug that blanked the graph at 1k scale with force flooring, velocity clamping, and finite guards
- made the graph modal near-full-screen, filling the viewport with a small margin
- moved the now-superseded `VaultGraph.tsx` experiment to `_to_delete/`
- verified `npm run lint` and `npm run build`; visually verified the small vault and the 1k demo across all four themes

Tauri desktop shell (Phase 1 scaffold):

- added `src-tauri/` for Tauri v2: Rust commands for pick-folder, list/read/write/create/rename/delete notes, plus a filesystem watcher that emits `vault-changed`
- added `tauri.conf.json`, a default capability, `build.rs`, and app icons
- added the frontend vault adapter `src/vault.ts` and the `src/useVault.ts` debounced-write hook; the browser still falls back to localStorage
- wired an `Open vault` action and disk persistence into `App.tsx`, gated by `isTauri` so browser behavior is unchanged
- added `@tauri-apps/api`, `@tauri-apps/plugin-dialog`, `@tauri-apps/cli`, and an `npm run tauri` script
- frontend verified with `tsc` and `vite build`; the Rust/native build runs locally with `npm run tauri dev`

## 2026-08-20 Tauri Build Verified, File Explorer, Rename/Delete, and Watcher Reconciliation

Finished Phase 1 (Tauri vault) end to end.

Build verification:

- installed the Rust toolchain (via Homebrew; this machine had none) and ran `npm install` for the new `@tauri-apps` packages
- `cargo check` on `src-tauri` compiled clean against current dependency versions — neither suspected v2 API spot (the dialog `blocking_pick_folder().to_string()` conversion, the `notify` watcher channel) needed a fix
- `npm run tauri dev` compiled the full debug binary and launched the desktop window; visually confirmed by the user

Frontend features:

- **File explorer**: added `src/noteTree.ts` (pure grouping of notes by folder path) and `src/NoteTreeView.tsx` (collapsible tree UI); replaced the flat sidebar note list. Verified with the 1k demo fixture — folders collapse/expand correctly and the active note stays highlighted.
- **Rename/delete on disk**: topbar gained inline rename (click pencil, edit in place, Enter/blur commits, Escape cancels) and delete (trash icon, native confirm) wired to `renameVaultNote`/`deleteVaultNote`. Added `renameWikilinks` in `src/noteUtils.ts` so every other note's `[[OldTitle]]` reference is rewritten to `[[NewTitle]]` on rename. Verified in the browser: renaming "Home" correctly updated the sidebar, path, and rewrote the backlink in another note; delete correctly removed the note and fell back to another active note.
- **Reconcile external edits**: wired the existing `watchVault` watcher into `App.tsx` — on a `vault-changed` event, notes refresh from disk except the currently-focused note, whose in-memory content is preserved so an in-flight edit is never clobbered. (Not end-to-end testable outside the native app; reasoned through and code-reviewed, needs a hands-on check by opening a real vault and editing a file externally.)
- Graph rebuild on load/change needed no new code — `buildKnowledgeGraph(notes)` already re-derives on every `notes` change.
- Verified `npm run lint`, `npm run build`, and `npm run check` (markdown + docs frontmatter) all pass.

Repo indexing plan:

- Decided the first external repo/project-indexing target: the Dakila Overland Controller repo (`OverlandLightingControllerV1`) at `/Users/paulbrettorozco/Sites/overland`, over the sibling `dakila-landing` static site — it is the substantive multi-language repo (ESP32 C++ firmware, Node.js backend, TypeScript/React Native mobile app, its own `docs/` and `AGENTS.md`).
- Added [[Repo Indexing]] as a new note with a phased plan (read-only source root → Markdown/manifest extraction → tree-sitter for C++/TypeScript → unified graph + MCP), and wired it into [[Product Roadmap]] (new Phase 6), [[Next Steps]], [[Backlog]], [[Decision Log]], [[Home]], and [[README]].

## 2026-08-20 Fixed a Main-Thread Deadlock in `pick_vault`, Verified the Watcher End to End

Real-world use surfaced a bug the build/compile checks couldn't catch: clicking "Open vault" hung the app with a spinning cursor.

Root cause (found by reading the actual crate source, not guessing): `pick_vault` was a plain (non-`async`) Tauri command, so Tauri dispatches its body synchronously on the same thread that handles the IPC call — the main thread on macOS. Inside it, `blocking_pick_folder()`'s own doc comment says it must not be called on the main thread. The main thread blocked waiting on the native panel, but showing/dismissing that panel needs the main thread's run loop free — a self-deadlock.

Fix: made `pick_vault` an `async fn`, matching the plugin's own documented usage pattern, so Tauri dispatches it off the main thread. Verified by the user: opening the vault now works.

Also verified, with the user driving the actual desktop window (something I can't automate — no tool here drives a native macOS window):

- rename and delete both work correctly on disk
- the `vault-changed` watcher reconciliation works end to end — confirmed with temporary diagnostic instrumentation (Rust-side `eprintln!` on each watcher stage, a temporary JS-side event counter) showing real fs events flow from `notify` through `app.emit` to the frontend refresh; instrumentation removed once confirmed

Phase 1 (Tauri vault) is now genuinely confirmed working end to end, not just compiling.

## 2026-08-20 UI Cleanup from a Live TODO Note

The user jotted requirements directly into a new `TODO.md` in the vault while this session was in progress (not through chat). Folded the UI-facing ones into the app, formatted the note to match vault conventions (frontmatter, added a heading), and left the repo-indexing requirements (repo picker, notes/repo/both graph toggle) open, pointing at [[Repo Indexing]].

Implemented:

- **Settings modal**: theme switcher and the "Load 1k demo" button moved out of the sidebar into a new Settings modal (same `.modalLayer` pattern as the graph modal), opened via a new gear icon next to the brand.
- **Removed the small inline graph preview** from the inspector panel (the mini SVG with dots/lines was hard to read anyway); kept a plain "Open graph" button plus the node count, and simplified the underlying computation (`previewNodeCount` replaces the old `positionGraph`-based preview layout, which is no longer needed without the mini visualization).
- Cleaned up now-unused CSS (`.previewNode`, `.graphHint`, and the `.graphPreview`-only rules, keeping the shared `.graphCanvas` rules used by the real graph modal).
- Verified visually in the browser: sidebar now shows only search, new note, open/switch vault, and the note tree; Settings modal opens with the theme swatches and demo button intact and working.
- `npm run lint`, `npm run build`, and `npm run check` all pass.

## 2026-08-20 Repo Indexing Phase 6a Implemented

Implemented the first working repo-source-root path:

- added a Tauri `pick_source_root` command and a single `index_source_root` command that recursively walks a selected repo/project folder, metadata only, with heavy generated/vendor folders ignored
- added `git_source_status` and git signature polling so Diamante re-indexes when HEAD or porcelain status changes
- added app-state persistence for both the opened vault and selected source root
- added `.diamante/graph.json` saving inside the opened vault, containing Graphify-compatible JSON plus Diamante metadata (`generatedAt`, vault path, source root, git state, stats)
- added `src/sourceRoot.ts` and `src/sourceGraph.ts` to convert source files into `folder`/`file` nodes and `contains` edges
- merged the repo graph with the vault note graph in `App.tsx`
- added a repo indexing button, visible loading/index status notice, repo status, and graph artifact path display
- added a Notes/Repo/Both origin switch in the full graph modal
- removed the now-unused `vis-network` dependency from `package.json` / lockfile
- added a Rust source-walk unit test proving metadata collection skips `node_modules`

Verified:

- `npm run lint`
- `npm run build` (passes; Vite still reports the existing large chunk warning)
- `cargo test`
- browser smoke check: graph modal opens, the canvas renders, and the Notes/Repo/Both switch is present
- `npm run tauri dev` launches the desktop shell successfully
- hands-on desktop verification by the user: repo indexing flow, graph behavior, and generated vault artifact are working

## 2026-08-20 UI/UX Simplification Pass

Ran a first Obsidian/Graphify-inspired simplification pass before continuing extraction work:

- collapsed the old right sync/dashboard column into an Obsidian-style bottom status bar
- moved secondary workspace actions into a compact icon action row: open/switch vault, index/switch repo, and open graph
- removed the duplicate graph panel from the inspector so there is one clear graph entry point
- kept primary state controls as text where clarity matters: editor mode, graph Local/Global, and graph Notes/Repo/Both
- converted graph node-type filters to icon buttons with accessible labels and hover titles
- locked the app shell to the viewport so large vaults scroll inside the note tree instead of stretching the whole page
- pruned stale sync-sidebar CSS after the layout change

Verified:

- `npm run lint`
- `npm run build` (passes; Vite still reports the existing large chunk warning)
- browser smoke check: main workspace has no page overflow, note tree scrolls internally, the graph modal opens from a single icon button, graph controls fit without horizontal overflow, and the canvas renders

Remaining UI work:

- compact/mobile viewport still needs a hands-on check in a resizable browser or Tauri window; the browser control wrapper did not apply a smaller viewport during this pass

## 2026-08-20 Theme Persistence and Note Tree Controls

Continued the workspace UX pass using [[Frontend]] as the local UI/UX guidance:

- added note-tree icon controls for sort by title and updated time; each sort button now toggles ascending/descending, alongside one expand/collapse-all folder toggle
- added drag/drop note moves: notes can be dragged onto folders or the root; desktop moves are handled by a new Tauri `move_note` command
- added three restrained workspace themes: Graphite, Paper, and Contrast
- hardened selected-theme persistence by saving it in both localStorage and Tauri app state
- normalized [[Frontend]] with vault frontmatter so it participates in docs validation
- updated [[TODO]] to mark drag/drop moves, note-tree controls, and theme persistence work done

Verified:

- `npm run lint`
- `npm run build` (passes; Vite still reports the existing large chunk warning)
- `cargo check`
- `npm run check`

## 2026-08-20 Folder Creation and Compact Graph Controls

Finished the remaining short UI TODOs from [[TODO]]:

- added empty-folder awareness to the vault tree, with Tauri `list_folders` and `create_folder` commands
- added a New Folder icon button to the note tree toolbar
- restored visible Open Vault and Open Repo workspace buttons, including the empty-vault state where no active note exists yet
- first vault load now collapses all folders, while newly created nested folders reveal their parent path
- changed expand/collapse-all into one folder-style toggle button
- changed note-tree sort buttons into toggles: title switches A-Z/Z-A, updated switches newest/oldest first
- made graph Local/Global and Notes/Repo/Both controls icon-only with hover titles and accessible labels
- moved graph node-type icons plus Depth, Edge, and Truth selects into a compact filter popover beside graph search

Verified:

- `npm run lint`
- `cargo check`
- browser smoke check: New Folder/expand/collapse controls render, graph filter popover opens, filter chips and selects are present, and graph controls do not overflow

## 2026-08-20 Docs Refresh and Next Plan

Updated the working docs after the UI repair pass:

- refreshed [[Backlog]], [[Next Steps]], [[Product Roadmap]], [[Repo Indexing]], and [[TODO]] to reflect visible Open Vault/Open Repo controls, empty-vault shell behavior, first-load folder collapse, New Folder reveal behavior, and sort-direction toggles
- updated the app README so it no longer describes Diamante as browser-only; it now documents the Tauri vault, repo source-root indexing, git-status polling, and `.diamante/graph.json`
- set the next implementation track as Phase 6b: repo Markdown/manifest extraction, then command palette and MCP/agent-facing graph commands

Next planned build sequence:

1. Add pure source extraction helpers for repo Markdown docs and `package.json` manifests.
2. Extend `buildSourceGraph` with `documents`, `references`, and `depends_on` edges while keeping file/folder containment as the base layer.
3. Save the richer Graphify-compatible artifact to `.diamante/graph.json`.
4. Add command palette actions for the main workspace commands and mirror them into the agent/MCP surface.

## 2026-08-20 Repo Indexing Phase 6b Implemented

Implemented the cheap repo extraction layer:

- extended the Tauri source-root walk to include bounded content only for extractable files: Markdown docs, `AGENTS.md`, and `package.json`
- kept ordinary source files metadata-only so the recursive walk stays lightweight
- extended `buildSourceGraph` with Markdown heading artifacts, `documents` edges, Markdown link `references` edges, wikilink references to matching vault notes, and package `depends_on` edges
- updated graph origin detection so repo-side artifact nodes remain visible under the Repo graph filter
- preserved Diamante node metadata in the Graphify-compatible `.diamante/graph.json` export
- upgraded the repo indexing notice to show how many docs/manifests were extracted
- refreshed [[Backlog]], [[Next Steps]], [[Product Roadmap]], [[Repo Indexing]], and [[TODO]] so Phase 6b is marked done

Verified:

- `npm run check`
- `npm run build` (passes; Vite still reports the existing large chunk warning)
- `cargo test`

Next planned build sequence:

1. Add the command palette for Open Vault, Open Repo, Open Graph, New Note, New Folder, sort toggles, folder expand/collapse, Load Demo, and Sync Preview.
2. Mirror graph/query/workspace actions into the MCP/agent surface.
3. Add source-root watch/rebuild parity with the vault watcher.
4. Add tree-sitter extraction for C++ and TypeScript/TSX.

## 2026-08-20 Preview Editing Toolbar and Code Blocks

Handled the latest [[TODO]] items:

- added a topbar Markdown-tools toggle for the active note
- added selection-aware formatting actions in `MarkdownEditor`: bold, italic, H1/H2, link, inline code, fenced code block, quote, bullet list, numbered list, and task list
- kept Preview readable while allowing edits there: when Markdown tools are open in Preview, Diamante shows the rendered preview plus a live source drawer underneath
- replaced plain preview code rendering with labeled, styled fenced code panels while keeping inline code styling
- cleaned minor markdownlint issues in [[Research]] and `AGENTS-GROK.md` that were blocking the docs check

Verified:

- `npm run lint`
- `npm run build` (passes; Vite still reports the existing large chunk warning)
- `npm run check`
- browser smoke check: Preview mode can show the Markdown toolbar, exposes 11 formatting controls, and opens the live source drawer while keeping Preview active

## 2026-08-20 Workspace Picker and Mode Switch Cleanup

Tightened the sidebar and editor controls after the latest UX pass:

- changed the Edit/Preview segmented control to icon-only buttons (`Code2` and preview eye), keeping screen-reader labels and hover titles
- changed the Open Vault/Open Repo buttons so their visible labels become the loaded vault or repo folder name after selection
- removed the redundant workspace target strip that sat between the picker buttons and graph action
- merged the repo index progress/stats notice into the bottom status bar beside git state, removing the remaining sidebar status panel before the note tree
- removed duplicate git branch/status text from the repo index item so the status bar reads as graph size, git state, index counts, artifact path
- updated [[TODO]] to mark the cleanup complete

## 2026-08-20 Push Checkpoint

Prepared the current UI/UX pass for push:

- app commit `9ba7645` covers Preview editing tools, styled code blocks, compact Edit/Preview icons, loaded vault/repo picker labels, and merged status-bar repo index stats
- vault notes now reflect the completed TODO items and session history for the UI pass
- `.diamante/graph.json` remains updated with the latest Graphify-compatible vault/repo graph artifact

## 2026-08-20 Sidebar and Status Bar TODO Batch

Handled the small UI cleanup batch from [[TODO]]:

- moved New Note under the Open Vault/Open Repo picker buttons
- moved Open Graph into the note-tree toolbar as the rightmost icon action
- hid the expand/collapse-all folder control when the vault tree has no folders
- made sort toggles smooth-scroll the note list back to the top
- replaced the prompt-based New Folder flow with a centered naming dialog seeded as `New Folder #`
- made the status bar horizontally scrollable on tight screens without visible scrollbars, while keeping `.diamante/graph.json` aligned to the right on wide screens

Remaining open TODOs from this batch:

- recent files menu combining vault notes and repo files
- Markdown image embeds that save image assets into `Images/` and insert links

## 2026-08-20 Note Tree Context Menu Batch

Handled the newest [[TODO]] items:

- moved New Note into the note-tree toolbar as a primary file-plus icon before New Folder
- added a right-click context menu for notes and folders with Rename, Delete, Duplicate, View in Finder, and Copy Path
- added Tauri vault item commands for folder rename/delete, generic note/folder duplicate, and native file-manager reveal
- kept browser/localStorage fallback behavior for rename/delete/duplicate/copy where possible, with View in Finder gated to the Tauri vault

Still open:

- recent files menu combining vault notes and repo files
- Markdown image embeds that save assets into `Images/`

Push checkpoint:

- app commit `c80b109` contains the note-tree toolbar move, context menu UI, and Tauri vault item commands
- verified with `npm run check`, `npm run build`, and `cargo check`

## 2026-08-20 Static Inspector Graph Preview

Restored the mini graph preview above the Backlinks panel, but with a stricter interaction model:

- renders a capped static SVG snapshot from the positioned graph data
- individual mini nodes and edges have no hover/selection behavior and ignore pointer events
- tapping/clicking the preview panel opens the full graph modal
- updated [[TODO]] to record the revised graph-preview contract

Follow-up:

- removed the mini graph preview again and replaced it with a compact Open Graph button above Backlinks

## 2026-08-20 Graph Filter Control Alignment

Refined the graph modal controls from the screenshot review:

- collapsed the node-type icon chip row into a single Node dropdown
- aligned Node, Depth, Edge, and Truth as one vertical dropdown set in the filter popover
- normalized graph modal icon buttons, search, and selects around a shared 38px control height
- updated [[TODO]] with the completed graph filter alignment pass

## 2026-08-20 Multi-Window Sessions

Added multi-session desktop support:

- added Tauri support for additional `session-*` webview windows
- expanded the default Tauri capability to allow `main` and `session-*` windows
- added a native File -> New Window menu item for opening another Diamante session
- removed the frontend New Window header button and command palette action so the behavior lives in the main window menu
- updated [[TODO]] with the completed multi-window item

## 2026-08-20 Graph Depth Range

- increased the full graph modal local-depth selector from 1-3 to 1-10 so deeper note/repo neighborhoods can be inspected without code changes

## 2026-08-20 Background Repo Symbol Indexer

Implemented the Graphify-style repo indexer pass from [[Repo Indexing]]:

- Tauri source indexing now maps all first-party files, reads bounded extractable content up to 200 KB in a second pass, and always excludes `node_modules`
- the frontend keeps the vault graph visible immediately, then stages repo indexing as mapped files followed by extraction batches of about 40 files
- added `src/extractSymbols.ts` for file-scoped functions, React components, classes/types, imports, and same-file inferred calls
- kept the existing graph schema: files/folders/packages/symbols/components plus `contains`, `defines`, `imports`, `calls`, `depends_on`, `documents`, and `references`
- the repo status chip can be clicked to re-index; the canceled `node_modules` package toggle was removed from Settings and app state
- command palette routes `functions`, `function Name`, and `fn Name` into repo graph symbol search
- MCP now exposes `search_symbols` over the saved Graphify-shaped graph artifact

Plan adjustment:

- loading a repo now starts with metadata only so file/folder nodes paint first
- once metadata is visible, Diamante starts a separate extraction pass and streams content-backed symbols into the same graph
- the note tree now shows Markdown filenames instead of note titles

## 2026-08-20 Vault-Scoped Repo Attachment

Adjusted workspace ownership:

- window title now follows the opened vault folder name
- the repo picker is disabled until a vault is open
- attached source repo is stored in the vault-local `.diamante/workspace.json`, not global app state
- opening a vault automatically restores and indexes that vault's attached repo when present

## 2026-08-20 Optional Function Indexing

Changed the repo indexer after the automatic symbol pass caused the app to hang/crash on load:

- opening or restoring a repo now loads metadata only: files, folders, manifests available from the previous stable graph surface
- function/symbol indexing is opt-in from the graph view via a code icon button
- local graph depth is capped at 3 again
- Tauri `index_source_root` now runs through `spawn_blocking` so file walking/content reads happen off the command/UI thread
- symbol graph construction runs in `src/sourceIndexWorker.ts`, keeping React usable while indexing is active
- if function indexing fails, the metadata repo graph remains available instead of emptying the graph
- the graph code button now pauses an active function index, terminates the worker, and invalidates stale completions; app unmount/close uses the same cleanup pattern

## 2026-08-20 Chunked Graph Artifact

Optimized the saved graph artifact for large repos:

- `.diamante/graph.json` is now a lightweight manifest with stats and chunk references
- graph payloads are saved under `.diamante/graph/` as Graphify-shaped node and edge chunks
- chunks are streamed to Tauri one file at a time instead of sent as one large IPC payload
- node chunks are grouped by vault, structure, packages, symbols, repo, etc.
- edge chunks are grouped by relation type such as `contains`, `defines`, `imports`, and `calls`
- MCP can read both the legacy single-file graph and the new manifest/chunk layout
- the legacy single-file writer remains as a fallback if chunk writing fails

## 2026-08-20 Graph Scale Draw Layer

Implemented the first [[Graph Scale]] pass after confirming the freeze is caused by drawing and physics, not indexing:

- kept the full merged vault plus repo graph intact for persistence, MCP, and query work
- added a capped draw graph for the graph modal so the canvas receives about 1,500 nodes max
- hid `contains` edges from the draw graph because folder containment overwhelms large repo views
- made graph-node clicks toggle a 1-hop expansion from the full filtered graph, capped around 80 neighbors with `+N more` in the details panel
- changed the status bar graph chip to report `view N / index M`

Next scale steps:

- cache frozen layout positions in `.diamante/layout.json`
- move full adjacency/path/impact work into a graph worker
- convert Stars to instanced points before drawing large repos there

## 2026-08-20 Independent Window Sessions

Fixed multi-window vault sessions:

- only the `main` Tauri window restores and saves the default vault path in app state
- `session-*` windows can open their own vault without changing the main window's saved vault path
- theme remains global across windows
- vault file-change watcher events are emitted only to the window that opened that vault, so another window does not refresh from unrelated vault changes

## 2026-08-20 Gitignore-Aware Repo Indexing

Added a repo indexing toggle for `.gitignore` handling:

- source walking now follows `.gitignore` by default instead of indexing ignored/generated files automatically
- Settings exposes an `Ignore .gitignore` checkbox for the attached repo
- the checkbox persists in vault-local workspace metadata and triggers a fresh re-index when changed
- optional function indexing uses the same flag, so metadata-only and symbol extraction stay aligned
- Rust tests now cover both default `.gitignore` filtering and the opt-out path

## 2026-08-20 Graph-Triggered Default Function Indexing

Adjusted the function-index workflow again:

- function indexing is now enabled by default for attached repos
- repo open still maps metadata only, keeping the app responsive while the workspace loads
- the first graph open starts background symbol extraction automatically when function indexing is enabled
- the function-index toggle moved out of the graph modal and into Settings beside the `.gitignore` toggle
- turning function indexing off stops future symbol extraction and reverts the repo graph back to metadata-only on re-index

## 2026-08-20 Graph Filter Popover Fix

Fixed the graph filter dropdown after it drifted into the graph details panel and could remain open:

- removed the stale graph-control grid column left behind after moving function indexing into Settings
- constrained the filter menu to the icon button width
- right-aligned the popover so it opens inward and stays inside the graph modal
- changed dismissal to capture-phase pointer handling plus Escape so clicks outside the menu close it reliably

## 2026-08-21 Commit-Only Repo Re-Index + UI Prefs

Optimized graph indexing and workspace chrome on Windows (`C:\Sites\diamante`):

- `GitRepoState.signature` is now `branch|HEAD` only (Rust `git_state`); dirty/untracked files no longer restart indexing
- git poll interval ~15s; status bar still shows dirty counts without thrashing the indexer
- function/symbol extraction remains once-after-map when the graph opens; commit-driven `indexRepo` can reset it
- stopped `indexRepo` from forcing graph origin/mode back to all/global
- persist Edit/Preview + graph mode/origin/depth/filters in `localStorage` (`diamante.uiPrefs`) and vault meta
- graph modal title uses repo folder name (else vault name)
- Explain panel wraps long filenames; no horizontal scrollbar; sitewide thin vertical scrollbars
- updated app README and vault notes ([[Repo Indexing]], [[Decision Log]])
- Windows release build via `npm run tauri build` (MSI + NSIS)

Verified:

- `npm run lint`
- Tauri dev shell launches after indexer change

## 2026-08-21 Graph Engine Toggle (2D / 3D)

Added a second graph renderer option on branch `new-graph-option` (`C:\Sites\diamante`):

- `graphEngine` preference: `canvas2d` (default, `GraphifyNetwork`) or `force3d` (`ForceGraph3DNetwork` + `react-force-graph-3d`)
- toolbar toggle (square = 2D, box = 3D) beside local/global and notes/repo origin
- same selection / path-highlight / Explain contract for both engines
- 3D chunk lazy-loaded; Three.js not in the initial app bundle
- persist `graphEngine` in `diamante.uiPrefs` and vault meta (Rust `VaultMeta.graph_engine`)
- fixed graph toolbar layout (flex instead of 4-column grid that broke after the extra icon group)
- app README + vault notes updated ([[Architecture]], [[Decision Log]], [[Graphify Tech Research]], [[Backlog]])

Verified:

- `npm run lint`
- `npm run build` (main ~900 kB gzip ~300 kB; 3D chunk separate)
- Tauri dev + live toolbar AX check

## 2026-08-21 Merged graph engine toggle to main

- Fast-forward merged `new-graph-option` into `main` (`1a3e731`)
- Follow-up on main: Windows `CREATE_NO_WINDOW` for git status polls (no console flash)
- Release build on main after merge

## 2026-08-21 Hermes MCP: diamante-dakila + diamante-docs

Wired two Hermes MCP stdio servers to `scripts/diamante-mcp.mjs`:

| Server | Vault |
| --- | --- |
| `diamante-dakila` | `C:/Users/webwi/Documents/Dakila` |
| `diamante-docs` | `C:/Users/webwi/Documents/Diamante` |

Both expose 11 tools (graph query + vault note writes). Hermes config updated; app `.mcp.json` mirrors the dual-server layout. New Hermes session required to load tools.

## 2026-08-21 Agent + human setup plans in docs vault

Captured the post-MCP-write product track in the Diamante docs vault:

- New canonical note: [[Agent and Human Setup]] (P0–P7: read tools, graph freshness, first-run MCP export, AGENTS.md contract, impact tools, Dakila extraction, git sync, no-Node MCP)
- Updated [[Next Steps]], [[Product Roadmap]], [[Backlog]], [[TODO]], [[Home]], [[README]], [[Command Palette and Agent Surface]], [[Decision Log]]
- Active milestone: finish agent loop + simple human setup before more chrome or in-app agent chat

## 2026-08-21 Distribution: versioning, OTA, landing page plans

Extended the docs track after agent/human setup:

- New [[Distribution Versioning and Updates]] — single version truth, release pipeline, Tauri OTA updater, signing
- New [[Landing Page]] — public marketing/download site TODO (hero, CTAs, requirements, honest claims)
- Linked from [[Agent and Human Setup]] P7, [[Next Steps]], [[Product Roadmap]], [[Backlog]], [[TODO]], [[Home]], [[README]]

## 2026-08-21 P0 — MCP note read surface

Shipped agent vault reads in app repo `scripts/diamante-mcp.mjs`:

- Tools: `list_notes`, `search_notes`, `read_note`
- Resources: `diamante://note/<path>` (plus existing graph stats/hubs)
- Path escape guarded via existing `safeJoin`
- Tests: `scripts/diamante-mcp-write.test.mjs` (write suite + new read/resource/escape cases) — pass

Docs: [[Agent and Human Setup]] P0 acceptance checked; [[Next Steps]] / [[Product Roadmap]] / [[Command Palette and Agent Surface]] updated. **Next:** P1 graph freshness after agent writes.

## 2026-08-21 P1 — Graph freshness after agent writes

Shipped in `scripts/diamante-mcp.mjs`:

- After vault writes: `vaultDirty` + cache reset; queries merge live vault note layer over artifact so new notes are visible immediately
- `graph_stats` adds `stale`, `reasons`, `artifactAgeMs`, `vaultNoteMtimeMs`, `sourceHead`, rebuild hint
- `rebuild_graph` persists merged Graphify-shaped `.diamante/graph.json` and clears stale
- Tests extended (write → stale → query sees note → rebuild → artifact + stats fresh)

**Next:** P2 first-run wizard + one-click MCP export.

## 2026-08-21 P2 — First-run wizard + MCP export

Shipped human setup surface in the app:

- `SetupWizard` multi-step: vault → optional repo → index tips → copy MCP JSON → done
- Auto-opens once when no vault and setup not completed (`localStorage`)
- Settings → Agent setup + command palette **Open setup wizard** / **Copy MCP config for agents**
- `src/mcpExport.ts` builds Hermes/Claude/Cursor-shaped `mcpServers` JSON with absolute vault path
- Rust `resolve_mcp_paths` locates `scripts/diamante-mcp.mjs` (dev path / exe-adjacent / `DIAMANTE_MCP_SCRIPT`)
- Smoke line reports graph/notes readiness + `node <script>`

**Next:** P3 app-repo `AGENTS.md` agent contract.

## 2026-08-21 P3 — App AGENTS.md agent contract

Rewrote app-repo `C:\Sites\diamante\AGENTS.md` as the agent contract:

- What Diamante is (app vs docs vault vs MCP)
- Windows-real paths for this machine
- Hard rules table (vault-only MCP writes, commit-driven re-index, freshness, etc.)
- Recommended agent workflow
- Full MCP tool + resource reference
- Codebase map + validation commands
- Pointers into the docs vault

P0–P3 agent/human setup track is complete for “done enough” core loop. **Next:** P4 higher-order graph tools, or user-chosen P5–P7.

## 2026-08-21 P4 — Higher-order graph MCP tools

Shipped in `scripts/diamante-mcp.mjs`:

- `explain_edge` — provenance-aware edge explanation between nodes (or by edge id)
- `impact_of` — multi-hop fan-out impact set
- `list_communities` — community sizes + samples
- Resource `diamante://graph/communities`
- Tests in `scripts/diamante-mcp-write.test.mjs`; app `AGENTS.md` tool table updated

**Next:** P5 deeper code extraction (Dakila), or P6/P7.

## 2026-08-21 Release 0.2.0

Cut app release **v0.2.0** after P0–P4:

- Version bumped in `package.json`, `tauri.conf.json`, `Cargo.toml`
- Settings → About shows version
- `CHANGELOG.md` added
- Windows MSI + NSIS via `npm run tauri -- build`
- GitHub Release `v0.2.0` with installers

## 2026-08-21 Next Steps: editor auto-format + code editor plan

Captured in docs vault [[Next Steps]] (and [[Backlog]] / [[TODO]]):

1. **Plain-text auto-format** — on paste/open of unformatted text, ask whether to format; show an Auto-format button when content looks unformatted; build on `src/MarkdownEditor.tsx`
2. **Lightweight code editor** — track [[Code Editor Implementation]] (CodeMirror 6 + Prettier + lint; Monaco deferred); target `src/components/code-editor/`
