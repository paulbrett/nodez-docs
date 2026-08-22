---
id: nodez-repo-indexing-phase-6a
title: Repo Indexing Phase 6a
type: architecture
status: active
created: 2026-08-20
updated: 2026-08-20
tags:
  - graph
  - repo-indexing
  - spec
---

# Repo Indexing Phase 6a

Implemented design for Phase 6a of [[Repo Indexing]]: a read-only "source root" over an external project folder (first target: the Dakila repo at `/Users/paulbrettorozco/Sites/overland/OverlandLightingControllerV1`), merged into the existing knowledge graph, with a picker, background indexing notice, git-status re-indexing, and a Notes/Repo/Both toggle in the graph UI.

## Goal

Pick an arbitrary repo folder, index its files in one recursive metadata-only Tauri call, show it in the graph as `file`/`folder` nodes with `contains` edges, alongside the notes graph, with a toggle to view notes only / repo only / both. Remember the chosen vault and source root across app restarts. Save the merged graph artifact inside the opened vault at `.nodez/graph.json` so agents can read one durable map of notes plus repo structure.

## Non-goals (explicit scope boundary)

- No content reading of repo files (no note-style editing of code).
- No tree-sitter/code extraction — no `imports`/`calls`/`defines` edges yet (Phase 6c).
- No full filesystem watcher for the source root yet. Phase 6a watches via git status polling: if HEAD or porcelain status changes, Nodez re-runs the metadata-only index. A future native source-root watcher can still land in Phase 6d.
- Exactly one source root at a time, matching the existing single-vault-path pattern. No cross-repo indexing.
- No content-derived code edges yet. The saved graph artifact is structural: vault note graph plus repo file/folder containment.

## Rust backend (`src-tauri/src/lib.rs`)

### New commands

- `async fn pick_source_root(app: AppHandle) -> Option<String>` — same `blocking_pick_folder()` pattern as `pick_vault`, **declared `async fn` from the start** (the `pick_vault` main-thread deadlock fixed this session showed why: a non-async command runs synchronously on the main thread, and `blocking_pick_folder()` must not be called there).
- `fn index_source_root(root: String) -> Result<SourceIndexResult, String>` — recursively walks `root`, skipping a hardcoded ignore-list by directory name, and returns **files only** (no folder entries — folders are derived client-side, see below), plus a git snapshot. No file content is read.
  - `RawSourceFile { path: String, extension: Option<String>, size_bytes: u64, modified_ms: u64 }` (camelCase over the wire, matching `RawNote`'s convention). `path` is root-relative with forward slashes, same convention as `RawNote.path`.
  - `GitRepoState { is_repo, root, branch, head, dirty, changed_count, untracked_count, signature, summary }` is built with `git -C <root>` and returned with the index. The `signature` combines HEAD and porcelain status so the frontend can detect meaningful git changes without watching every file.
  - Ignore-list (checked by directory name at every level, extending the existing `is_ignored()` helper): `node_modules`, `.git`, `.nodez`, `build`, `dist`, `target`, `ios`, `android`, `.expo`, `__pycache__`, `.venv`, `venv`, `coverage`, `.next`, `.vite`.
  - No path-traversal guard needed (unlike the vault's `safe_join`) — this is read-only with no write/rename/delete commands over the source root, so there is nothing for a traversal to escalate to.
- `fn git_source_status(root: String) -> GitRepoState` — lightweight status call for polling after an index.
- `fn load_app_state(app: AppHandle) -> Result<AppState, String>` / `fn save_app_state(app: AppHandle, state: AppState) -> Result<(), String>` — `AppState { vault_path: Option<String>, source_root: Option<String> }`, persisted as JSON at `<app_data_dir>/state.json` (via `tauri::Manager::path().app_data_dir()`), creating the directory on first save if missing. A missing file on load returns a default empty `AppState`, not an error — that's the expected first-run case.
- `fn save_graph_index(vault: String, content: String) -> Result<String, String>` — writes the generated merged graph to `<vault>/.nodez/graph.json`.

No new Cargo dependencies — `serde`/`serde_json` are already present, and `std::fs` is sufficient for the walk (the vault's own `collect()` already hand-rolls the same recursion style).

## Frontend

### `src/sourceRoot.ts` (new, mirrors `vault.ts`)

- `pickSourceRoot(): Promise<string | null>`
- `indexSourceRoot(root: string): Promise<SourceIndexResult>` → `{ root, files, git, indexedAtMs }`
- `gitSourceStatus(root: string): Promise<GitRepoState>`
- `loadAppState(): Promise<{ vaultPath: string | null; sourceRoot: string | null }>`
- `saveAppState(state: { vaultPath: string | null; sourceRoot: string | null }): Promise<void>`
- `saveGraphIndex(vault: string, content: string): Promise<string>`

### `src/sourceGraph.ts` (new)

- `buildSourceGraph(files: SourceFile[], root: string): KnowledgeGraph` — pure function:
  - one `file` node per entry (`id = "file:" + path`, `label` = filename, `path`/`sourcePath` = full path).
  - `folder` nodes derived from directory segments, same recursive split-and-group approach `noteTree.ts`'s `buildNoteTree` already uses for the vault — but building graph nodes/edges instead of a UI tree.
  - `contains` edges (folder → child file or subfolder), `provenance: "extracted"`, `confidence: 1`.

### `src/types.ts`

- Add `"contains"` to `GraphEdgeType`.

### `src/graph.ts`

- Add `mergeGraphs(...graphs: KnowledgeGraph[]): KnowledgeGraph` — plain concatenation of nodes/edges. No id collisions to resolve: vault nodes are namespaced `note:`/`tag:`, repo nodes `file:`/`folder:`.
- Add `nodeOrigin(node: KnowledgeGraphNode): "vault" | "repo"` — `"repo"` when `node.type` is `file`/`folder`/`symbol`/`package` (future-proofed for Phase 6c's symbol/package nodes), else `"vault"`.
- Extend `GraphFilters` (currently `{query?, nodeTypes?: Array<"all" | GraphNodeType>, edgeTypes?: Array<"all" | GraphEdgeType>, provenances?: Array<"all" | GraphProvenance>}`) with `origins?: Array<"all" | "vault" | "repo">`, matching the exact existing pattern for the other three fields. In `filterGraph`'s body: `const origins = new Set(filters.origins?.length ? filters.origins : ["all"]); const matchesOrigin = origins.has("all") || origins.has(nodeOrigin(node));`, folded into the same node-visibility check the function already computes (alongside `matchesType`/`matchesQuery`). This is the toggle's actual mechanism — not a separate filtering pass.

### `src/App.tsx`

- New state: `sourceRoot: string | null`, `sourceFiles: SourceFile[]`, `sourceGit: GitRepoState | null`, `isSourceIndexing`, `sourceIndexNotice`, `graphIndexPath`, `graphOrigin: "all" | "notes" | "repo"`.
- Sidebar: new button next to "Open vault"/"Switch vault" (same `isTauri` gate) — "Index repo" / "Switch repo", calling `pickSourceRoot()` → `indexSourceRoot()` → sets `sourceRoot`/`sourceFiles`/`sourceGit`.
- Sidebar notice: while indexing, show an "Indexing" notice; after indexing, show file count and git summary.
- `knowledgeGraph` becomes `mergeGraphs(buildKnowledgeGraph(notes), sourceRoot ? buildSourceGraph(sourceFiles, sourceRoot) : { nodes: [], edges: [] })`.
- Graph modal: a three-way Notes/Repo/Both toggle next to the existing Local/Global mode switch. `graphOrigin` maps to `filterGraph`'s new `origins` option: `"all"` → omitted (no origin filtering), `"notes"` → `["vault"]`, `"repo"` → `["repo"]`. Passed into the existing `filterGraph(...)` call in `filteredKnowledgeGraph`'s `useMemo`, alongside the current `nodeTypes`/`edgeTypes`/`provenances`.
- On mount (`isTauri` only): `loadAppState()` — if `vaultPath` present, auto-`listVaultNotes` + `setVaultPath` (skip the picker); if `sourceRoot` present, auto-`indexSourceRoot` + `setSourceRoot`.
- On `vaultPath`/`sourceRoot` change (`isTauri` only): `saveAppState({ vaultPath, sourceRoot })`.
- On `sourceRoot` with a git repo: poll `git_source_status` every 7 seconds. If the signature differs from the previous index, re-run `index_source_root` and refresh the repo graph.
- On `knowledgeGraph` changes with an opened vault: debounce 800 ms, then write `.nodez/graph.json` with Graphify-compatible JSON plus metadata (`generatedAt`, `vaultPath`, `sourceRoot`, `sourceGit`, `stats`).

## Data flow

Click "Index repo" → native folder picker (async command, off the main thread) → JS receives the root path → `indexSourceRoot(root)` invokes the Rust walk → Rust returns a flat file list plus git state (metadata only, noise directories excluded) → JS derives folder nodes and `contains` edges via `buildSourceGraph` → merged into the same `knowledgeGraph` the notes already populate → all existing graph UI (filters, local/global mode, path finder, Explain panel) works over the combined graph unchanged → the new Notes/Repo/Both toggle narrows the visible graph by node origin before rendering → the merged graph is saved at `.nodez/graph.json` in the opened vault.

## Error handling

- `index_source_root` on an unreadable/missing root → `Err(String)`, surfaced in the sidebar notice and logged to the console.
- `load_app_state` with no state file yet (first launch) → default empty state, not an error.
- A persisted `vaultPath`/`sourceRoot` that no longer exists on disk (moved/deleted since last launch) → the subsequent `list_notes`/`index_source_root` call fails, caught like any other listing error; the app falls back to its unopened state rather than crashing.

## Testing / verification

- Verification: `npm run lint`, `npm run build`, and `cargo test` pass. Rust includes a unit test for the ignore-list/file-walk logic against a small temp fixture directory that includes a `node_modules` folder, proving it is excluded.
- Browser smoke check passed: the graph modal opens, the canvas has real dimensions, and the Notes/Repo/Both origin switch is visible.
- Hands-on desktop verification passed via `npm run tauri dev`: the app launches, the repo indexing flow works, the graph behaves correctly, and the vault graph artifact is generated.

Related: [[Repo Indexing]], [[Unified Knowledge System]], [[Graphify Tech Research]], [[Architecture]], [[Decision Log]]
