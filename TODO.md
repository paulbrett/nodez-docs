---
id: diamante-todo
title: TODO
type: backlog
status: active
created: 2026-08-20
updated: 2026-08-20
tags:
  - backlog
  - repo-indexing
  - ui
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
- must also have command pallete.. these commands must also be added to exposed skills for AI agents.. similar to ibsidian skills,, and MCP to provide the graph and notes to AI agents — next: add command palette, then mirror commands and graph queries into the MCP/agent surface
- add tooltip or name of the button label if it has no label — done for current icon-only controls via `title` plus `aria-label`
- persist selected theme and add more themes — done: selected theme persists in localStorage and Tauri app state; added Graphite, Paper, and Contrast themes
- add an "add folder" option for notes; refine the collapse/expand icons — done: note tree has New Folder plus one folder-style expand/collapse toggle; newly created nested folders reveal their parent path
- in graph view, make the toggles icon-only; move the node-type icons and depth/edge/truth controls into a compact dropdown beside the search bar — done: graph mode/origin are icon-only, and node/depth/edge/truth controls live in the filter popover

- user can still edit in preview,, add toggable toolbar for editing markdown file in there — done: topbar Markdown-tools toggle opens a formatting toolbar; in Preview it also opens a live source drawer under the rendered preview
- show code formating to code blocks — done: fenced preview code blocks now render as labeled, styled code panels; toolbar includes inline-code and code-block insert actions
- change edit/preview to icon-only and show loaded vault/repo names on the picker buttons — done: the mode switch now uses code/preview icons, Open Vault/Open Repo become the active folder names after loading, and the duplicate workspace target strip before the graph action is removed
- merge sidebar repo-index stats into the status bar — done: repo indexing/loading/extracted-file status now sits in the bottom status bar next to git state instead of taking a sidebar panel; git branch/status is shown only once
