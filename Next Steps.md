---
id: diamante-next-steps
title: Next Steps
type: roadmap
status: active
created: 2026-08-20
updated: 2026-08-20
tags:
  - roadmap
  - planning
---

# Next Steps

A prioritized plan for what comes after the graph system and the Tauri shell scaffold. Ordered by dependency: finish the desktop vault first, since sync, extraction, and the agent workflow all assume real files on disk.

## Phase 1 (Tauri vault) — done

The shell builds and runs with a real Rust toolchain (`cargo check` and `npm run tauri dev` both verified end to end; neither of the two suspected v2 API spots needed a fix against the current dependency versions). All the following landed:

- **File explorer.** The vault renders as a collapsible folder tree in the sidebar (`src/noteTree.ts` groups notes by directory, `src/NoteTreeView.tsx` renders it).
- **Full note lifecycle on disk.** Rename and delete are wired to `rename_note` / `delete_note` from the topbar, including rewriting `[[wikilinks]]` in every other note that referenced the old title (`renameWikilinks` in `src/noteUtils.ts`).
- **Reconcile external edits.** The `vault-changed` watcher refreshes notes from disk, skipping the currently-focused note's content so an in-flight edit is never clobbered.
- **Graph rebuild on load/change** falls out for free — `buildKnowledgeGraph(notes)` already re-derives on every `notes` change, so it follows the vault automatically once the above keep `notes` in sync with disk.

## Phase 6a — repo source root done

Diamante can now choose a second read-only source folder, index it with a single metadata-only recursive Tauri command, and merge it into the graph as `folder`/`file` nodes with `contains` edges.

- The sidebar shows an indexing notice while the repo folder is loading.
- Git repo state is captured with each index and polled every 7 seconds; when HEAD or porcelain status changes, the repo graph re-indexes.
- The graph modal includes a Notes/Repo/Both origin switch.
- The merged graph artifact is saved in the opened vault at `.diamante/graph.json` for agents to read.

## Phase 6b — cheap repo extraction done

Diamante now enriches the repo graph without jumping to tree-sitter yet. This keeps the graph useful for agents quickly and avoids overbuilding parser infrastructure before the data model proves itself.

- The source-root walk still captures metadata for every non-ignored file, but now includes bounded text content for Markdown docs, `AGENTS.md`, and `package.json`.
- Markdown headings become repo `artifact` nodes, with `documents` edges from the source file.
- Markdown links and wikilinks become `references` edges to repo files and matching vault notes where possible.
- Package manifests become `package` nodes with `depends_on` edges for dependencies, dev dependencies, peer dependencies, and optional dependencies.
- Extracted node metadata is preserved in the Graphify-compatible `.diamante/graph.json` artifact.
- The repo indexing notice now reports how many docs/manifests were extracted.

## Next: Command Palette + Agent Surface

After Phase 6b, expose the useful actions instead of hiding them behind the UI:

- Add a command palette for Open Vault, Open Repo, Open Graph, New Note, New Folder, Sort Title, Sort Updated, Expand/Collapse Folders, Load Demo, and Sync Preview.
- Mirror those commands into the agent-facing capability list so AI agents can perform the same workspace operations intentionally.
- Start aligning the MCP surface with the graph artifact: `search_nodes`, `get_neighbors`, `find_path`, `explain_edge`, `impact_of`, `list_communities`, `most_connected`, and graph-backed resources for notes/files.

## Then: Phase 2 — GitHub sync

Turn the sync panel from a preview into real git, in Rust.

- Detect the git repo, show status (ahead/behind, changed files).
- Pull, stage, commit with a generated message, push.
- A conflict screen that shows both versions and offers guided resolution (never silently overwrite — per the sync decision).
- Rebuild the graph after a successful pull.

## Parallel track: MCP + extraction (the Dakila workflow)

- **Harden the MCP server.** Align its tool surface with the research note: `search_nodes`, `get_neighbors`, `find_path`, `explain_edge`, `impact_of`, `list_communities` / `most_connected`, `get_subgraph`. Expose notes and files as MCP resources so an agent can read what a query points to.
- **Deeper extraction.** Add tree-sitter (WASM in browser, native under Tauri) to emit real code edges — `imports`, `calls`, `defines` — with `inferred` provenance. This is what makes the graph span docs and code beyond the Markdown/manifest layer now in place.
- **Rebuild-after-sync** ties the graph, MCP, and Phase 2 together: pull → re-extract → re-serve.
- **External repo/project indexing** now has Phase 6a and 6b foundations in place. Next repo-indexing work should add tree-sitter code edges. See [[Repo Indexing]] — first target is the Dakila repo (`OverlandLightingControllerV1`) at `/Users/paulbrettorozco/Sites/overland`, chosen for its real mix of C++ firmware, Node.js backend, and TypeScript/React Native mobile code plus its own docs folder.

## Graph UX polish

- Declutter labels at scale (currently label-on-hover/hub only); optional community hulls; make the provenance filter more visible.
- Consider persisting node positions so the layout is stable across opens.
- Add a minimap or "zoom to fit / zoom to selection" controls.
- Compact/mobile viewport needs hands-on Tauri/browser review after the workspace picker and status bar changes.

## Phase 3 knowledge features

- Command palette, document outline, backlink context snippets, attachments, and surfacing unresolved links.

## Cleanup and tech debt

- Delete the `_to_delete/` folder once everyone is comfortable losing the superseded graph experiment. The now-unused `vis-network` dependency has been removed.
- Code-split the graph renderer so the main bundle shrinks (the build warns it is over 500 kB).
- Add graph performance measurements using the 1,000-node demo fixture.
- Add a focused test for note-tree first-load collapse and new-folder reveal behavior.

## Phase 5 polish and distribution

- Generate the full icon set with `npm run tauri icon`, then produce signed/notarized macOS builds and Windows/Linux installers.
- Keyboard shortcuts, export, and a mobile-responsive layout.

Related: [[Product Roadmap]], [[Architecture]], [[GitHub Sync]], [[Tauri Desktop Shell]], [[Unified Knowledge System]], [[Graphify Tech Research]], [[Repo Indexing]]
