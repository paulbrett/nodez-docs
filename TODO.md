---
id: diamante-todo
title: TODO
type: backlog
status: active
created: 2026-08-20
updated: 2026-08-22
tags:
  - backlog
  - repo-indexing
  - ui
  - agents
---

# TODO

- must be compatible with Obisidian file system and behaves like one
- must be able to choose which repo to index similar to graphify — done: added the Tauri source-root picker and metadata-only repo index, see [[Repo Indexing Phase 6a]]
- when making the graph: when watched/indexed repo is available, it can also be shown as graph with the notes but can be toggled: notes only, repo only, or both — done: added the graph origin switch and git-status re-indexing, see [[Repo Indexing Phase 6a]]
- create a setting page.. move the theme switcher there and other stuff that is not needed right away and optional.. make the UI distractionless..
  remove the small graph preview.. retain the button — done: added a Settings modal (theme switcher + "Load 1k demo"), removed the small inline graph preview from the inspector panel, and moved graph access into the sidebar icon action row
- simplify secondary actions into icon buttons — done for vault/repo/graph workspace actions, note rename/delete, sync preview, settings, graph node filters, and modal close controls
- enable drag and drop notes inside folders of notes.. and it autmatically resolve the links — done: notes can be dragged onto folders or the root; title-based wikilinks stay valid because the note title does not change
- add shortcut button icons for notes sorting and folders expand collapse — done: note tree now has icon controls for title/updated sorting, each sort button toggles ascending/descending, first vault load collapses folders, and there is one expand/collapse-all toggle
- must also have command pallete.. these commands must also be added to exposed skills for AI agents.. similar to ibsidian skills,, and MCP to provide the graph and notes to AI agents — partial: command palette + MCP graph query + vault writes + **note reads** (`list_notes`/`search_notes`/`read_note`) done; remaining [[Agent and Human Setup]] (P1 freshness, P2 one-click MCP export, P3 AGENTS.md contract)
- add tooltip or name of the button label if it has no label — done for current icon-only controls via `title` plus `aria-label`
- persist selected theme and add more themes — done: selected theme persists in localStorage and Tauri app state; added Graphite, Paper, and Contrast themes
- add an "add folder" option for notes; refine the collapse/expand icons — done: note tree has New Folder plus one folder-style expand/collapse toggle; newly created nested folders reveal their parent path
- in graph view, make the toggles icon-only; move the node-type icons and depth/edge/truth controls into a compact dropdown beside the search bar — done: graph mode/origin are icon-only, and node/depth/edge/truth controls live in the filter popover

- user can still edit in preview,, add toggable toolbar for editing markdown file in there — done: topbar Markdown-tools toggle opens a formatting toolbar; in Preview it also opens a live source drawer under the rendered preview
- show code formating to code blocks — done: fenced preview code blocks now render as labeled, styled code panels; toolbar includes inline-code and code-block insert actions
- change edit/preview to icon-only and show loaded vault/repo names on the picker buttons — done: the mode switch now uses code/preview icons, Open Vault/Open Repo become the active folder names after loading, and the duplicate workspace target strip before the graph action is removed
- merge sidebar repo-index stats into the status bar — done: repo indexing/loading/extracted-file status now sits in the bottom status bar next to git state instead of taking a sidebar panel; git branch/status is shown only once

- add recent files menu - a combination of vualt and repo
- when toggling sort - slide back up to top — done: note tree sort controls now smooth-scroll the list back to the top after changing sort mode/direction
- show only collapse/epand icon if there's a folder — done: the expand/collapse-all control is hidden until the tree has folders
- move the graph button right side of the new folder etc group.. rightmost — done: Open Graph now sits at the far right of the note-tree toolbar
- when making new folder, show the main editor a centered menu with 'New Folder #' and user can edit the folder name — done: New Folder opens a centered naming dialog seeded with the next available `New Folder #`
- move the 'new note' button under the open and repo buttons — done: New Note now sits directly under the Open Vault/Open Repo controls
- md files can embed images, the images will automatically saved to Images folder and linked in the md file
- status bar fix alignment of the graph.json.. make it horizontally scrollable if screen width is tight but do not show scrollbars — done: graph artifact status aligns right on wide screens, and the status bar scrolls horizontally on tight widths with hidden scrollbars

- make the 'new note' a plus icon before the add new folder icon.. same size but primary color — done: New Note is now a primary file-plus icon at the start of the note-tree toolbar, before New Folder
- when right clicking a file or folder there should be a contect menu 'Rename','Delete','Duplicte','View in Finder', 'Copy Path' — done: notes and folders now have a right-click menu with Rename, Delete, Duplicate, View in Finder, and Copy Path
- bring back graph preview on top of Backlinks panel as static only — done: inspector shows a non-interactive mini graph snapshot above Backlinks; tapping it opens the graph modal
- remove graph preview and replace with a button — done: inspector now shows a compact Open Graph button above Backlinks instead of the mini graph canvas
- graph page filter controls should be one dropdown set and aligned — done: node type is now a dropdown beside Depth, Edge, and Truth; graph buttons/selects use a shared 38px control height
- make app multi session / allow new window — done: Tauri can open additional `session-*` windows from the native File -> New Window menu
- each window should keep an independent vault session — done: only the main window owns the restored default vault path, session windows no longer overwrite global vault state, and vault watcher events are scoped to the window that opened the vault
- implement Graphify-style background repo indexer with symbols — done: the Tauri source walk maps first-party files metadata-only first, the frontend starts extraction afterward and streams batches, `src/extractSymbols.ts` adds defines/imports/same-file calls, `node_modules` stays excluded, the status chip re-indexes, and MCP exposes `search_symbols`
- file tree should show Markdown filenames, not note titles — done
- window title should show the opened vault name — done
- repo picker should require an opened vault and store the attached repo under vault metadata — done: repo attachment now lives in `.diamante/workspace.json` inside the vault
- make function indexing optional because automatic symbol extraction can hang/crash large repos — done: repo open now maps metadata only, function indexing is enabled by default but starts only when the graph opens, the toggle lives in Settings, native walking uses a blocking worker thread, and graph construction runs in a Web Worker
- allow pausing function indexing and cancel it on app close — done: turning off function indexing in Settings stops background symbol work, metadata re-index stays available, and app unmount/close still terminates the worker
- chunk large graph artifacts — done: `.diamante/graph.json` is now a lightweight manifest and graph payloads are written under `.diamante/graph/` by node/edge groups
- optimize large graph rendering without capping the index — first pass done: see [[Graph Scale]]; repos over 1,000 indexed files render a capped draw graph, hide `contains` edges, expand 1 hop on node click, and the status bar reports `view N / index M`; smaller repos keep the previous full graph behavior
- add option to ignore `.gitignore` while keeping it respected by default — done: repo indexing now follows `.gitignore` by default via the Rust walker, and Settings exposes an `Ignore .gitignore` toggle that persists with the attached repo metadata and re-indexes immediately
- fix graph filter dropdown placement and dismissal — done: the filter popover now anchors inside the graph modal under the filter icon and closes on outside pointer clicks or Escape

## Agent + human setup (2026-08-21)

Canonical plan: [[Agent and Human Setup]]. Summary checklist:

- P0–P4: done (MCP reads, freshness, wizard/export, AGENTS.md, impact/explain/communities)
- P5: deeper code extraction — regex pass landed; tree-sitter optional
- P6: real GitHub sync — v1 landed
- P7: **landing + OTA skeleton landed** ([[Landing Page]], [[Distribution Versioning and Updates]]); remaining: Pages enable, signed tag feed, Authenticode, no-Node MCP, icons

## Editor UX (2026-08-21)

- Plain-text / paste auto-format with ask-first prompt + Auto-format button — **first cut landed** — [[Next Steps]]
- Lightweight code editor plan — **MVP landed** — [[Code Editor Implementation]]
