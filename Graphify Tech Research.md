---
id: diamante-graphify-tech-research
title: Graphify Tech Research
type: research
status: active
created: 2026-08-19
updated: 2026-08-20
tags:
  - graphify
  - graph
  - mcp
  - storage
  - visualization
  - research
---

# Graphify Tech Research

Research into the three technical pillars Diamante wants from Graphify: how the graph is **stored**, how it is **served to agents over MCP**, and how it is **visualized** — followed by a prioritized view of what is doable in Diamante now versus what needs the [[Tauri Desktop Shell]].

> Sourcing note: the original draft was written while external web access was unavailable. On 2026-08-20, the Graphify implementation details below were checked against the official Graphify docs and open-source code.

## The Graphify pattern

Graphify-class tools all share the same three-stage pipeline:

1. **Extract** — parse source and docs into typed nodes and edges.
2. **Store** — persist that graph in a queryable index, with provenance on every edge.
3. **Serve + visualize** — expose the graph to agents (MCP) and to humans (interactive graph UI).

Diamante already has the vocabulary for this in [[Unified Knowledge System]] (node types, edge types, `extracted`/`inferred`/`manual` provenance). This note is about the machinery underneath.

## 1. Storage layer

### Extraction

How the raw graph gets built. In priority order for a Diamante-shaped vault + repo:

- **Markdown / YAML (already doable today).** Wikilinks, frontmatter, tags, and headings are parsed with a plain Markdown AST. This is the vault index Diamante already computes in the prototype. Provenance = `extracted`.
- **Tree-sitter for code (the standard).** Tree-sitter is the near-universal choice for code graphs: fast incremental parsers, one grammar per language, produces a concrete syntax tree you walk to emit `defines`, `imports`, `calls`, `references` edges. Bindings exist for JS/TS (`web-tree-sitter`, WASM) and Rust (`tree-sitter` crate). WASM build runs in the browser today; native runs under Tauri.
- **LSP / language servers (higher fidelity, later).** For accurate cross-file symbol resolution (go-to-definition, find-references), a language server gives better `calls`/`references` edges than tree-sitter alone. Heavier to wire up; a Phase 4+ enhancement. Provenance = `inferred`.
- **Package manifests.** `package.json`, `Cargo.toml`, lockfiles → `depends_on` edges. Cheap, high value, `extracted`.

Practical rule from the vault's own boundary: the indexer rebuilds from files and must not require a database to exist. So extraction output should be a plain, regenerable artifact.

### Where the graph lives

Four realistic options, roughly in order of when Diamante should adopt them:

- **In-memory + JSON snapshot (now).** Build the graph in memory (JS objects / `Map`s), persist a `graph.json` under `.diamante/`. Zero dependencies, works in the browser prototype, diff-able in git, matches "rebuild from files, no database." Good to a few thousand nodes. This is the right Phase 0/1 store.
- **SQLite (the pragmatic middle).** A single-file DB with `nodes` and `edges` tables. Recursive CTEs handle path/neighbor queries; FTS5 handles fast search; it is still just one file you can commit or gitignore. Available in the browser via `sql.js`/`wa-sqlite` (WASM) and natively under Tauri via `rusqlite`/`tauri-plugin-sql`. This is the recommended target store once the vault outgrows JSON.
- **Embedded property-graph DB (the "real graph" option).** **KuzuDB** remains a candidate for Diamante if graph queries outgrow JSON/SQLite. The verified Graphify open-source package currently treats `graphify-out/graph.json` as the primary downstream artifact, with optional exports to systems such as GraphML, Obsidian, Neo4j, and FalkorDB.
- **Neo4j / hosted graph DB — explicitly avoid.** Server-based, conflicts with the local-first, no-proprietary-service decisions in [[Decision Log]]. Only revisit if Diamante ever grows a multi-user backend.

### Schema (aligns with Unified Knowledge System)

Store two tables/collections:

- `nodes`: `id`, `type` (note/file/folder/symbol/package/decision/feature/component/tag/artifact), `label`, `path`, optional `line`/`heading`, `meta`.
- `edges`: `src`, `dst`, `type` (links_to/imports/calls/defines/depends_on/references/documents/decides/supersedes/related_to), `provenance` (extracted/inferred/manual), `confidence` (0–1), `source_path`, optional `line`/`heading`.

The provenance + source-reference columns are the non-negotiable part — they are what let the graph "explain" an edge and point back to truth, per the [[Architecture]] boundary that the graph must never invent implementation truth.

### Incremental indexing

- Hash each file (content hash in the index). On a filesystem-watcher event, re-parse only changed files, remove their old outgoing edges, re-emit new ones, recompute affected backlinks. This matches [[Backlinks and Indexing]].
- Full rebuild stays available as a fallback and for "rebuild graph after git sync" (Phase 4 roadmap item).
- Keep the whole store under `.diamante/` so it is app metadata, not note content.

## 2. MCP data-serving layer

"Provide data via MCP" means running a small **Model Context Protocol server** that exposes the graph so an agent (Claude, Codex, etc.) can query relationships *before* reading many files — the core of the Dakila workflow loop in [[Unified Knowledge System]].

### Shape of an MCP server

- MCP servers expose two things: **tools** (callable functions the agent invokes) and **resources** (readable documents/URIs the agent can pull). For a code graph, **tools** carry the weight — the agent asks questions; resources are useful for exposing individual notes/files by URI.
- **Transport:** for a local desktop app, **stdio** is the natural transport — the Tauri app (or a sidecar binary) launches the MCP server as a subprocess. A local HTTP/SSE transport is the alternative if the server needs to be shared across processes.
- **Implementation:** an official MCP SDK exists for **TypeScript** and **Python** (and community Rust). Because Diamante's index logic will already be in TS (prototype) and later Rust (Tauri), a TS MCP server that reads the same `graph.json`/SQLite store is the lowest-friction start; a Rust sidecar is the native end-state.

### Concrete tool set to expose

This is the "query/path/explain" surface from the vault, made concrete:

- `search_nodes(query, type?, limit?)` — text/tag search over nodes.
- `get_neighbors(node_id, edge_types?, depth?)` — local graph around a node (powers "local graph mode").
- `find_path(from_id, to_id, max_hops?)` — path tracing between two nodes (note ↔ file ↔ symbol ↔ decision).
- `explain_edge(src, dst)` — return the edge's type, provenance, confidence, and source file/line so the agent can cite it.
- `impact_of(node_id)` — reverse-dependency / blast-radius query ("what references this?").
- `list_communities()` / `most_connected(limit)` — subsystem clusters and hub nodes.
- `get_subgraph(filter)` — a scoped export (by tag, folder, node/edge type, provenance) the agent can reason over without reading files.

Each result should carry provenance + source path so the agent can fall through to reading the exact file/line — never treat the graph as ground truth for implementation behavior (the vault's conflict-resolution rule).

### Resources

- Expose notes and source files as MCP resources (`diamante://note/<id>`, `diamante://file/<path>`) so the agent can read the specific artifact a query points to. This keeps `AGENTS.md`'s "query the graph, then read the files it points to" loop tight.

## 3. Graph visualization

### Library choice (the key decision)

Pick by scale, because that dictates the rendering technology:

- **`vis-network` (implemented Graphify-compatible default).** Graphify's open-source HTML exporter uses `vis-network@9.1.6` with ForceAtlas2-style physics, dot nodes, degree-based sizing, community colors, search, click-to-inspect, arrowed edges, and confidence-styled dashed edges. Diamante now uses the same renderer family through `src/GraphifyNetwork.tsx` so the full graph modal behaves like Graphify while retaining Diamante-specific filters and Explain panels.
- **`react-force-graph` (superseded in Diamante).** React wrapper over `d3-force`; useful for custom canvas rendering, but it is no longer the Diamante default because the current goal is close Graphify visual compatibility.
- **`Cytoscape.js`.** Mature graph library with strong layout algorithms (including hierarchical/`dagre`), rich styling, and built-in graph analysis (centrality, shortest path). Excellent if you want layout + analysis in one package; slightly heavier API. Good "serious graph UI" option for the full modal.
- **`Sigma.js` + `graphology` (for scale).** WebGL renderer built for large graphs (tens of thousands of nodes). `graphology` gives you the data model plus algorithms — Louvain **community detection**, centrality/**most-connected**, shortest path — which map directly onto the vault's "communities" and "hub nodes" features. This is the endgame if vaults get big; possibly overkill early.
- **`d3-force` raw.** Maximum control, most code. Only if the wrappers get in the way.

Recommendation: **use `vis-network` for the Graphify-compatible graph surface now, and pull in `graphology` later only as the analysis engine** (communities, centrality, pathfinding) feeding whichever renderer Diamante keeps. That splits "how it looks" from "what it computes" cleanly.

### Rendering technology by scale

- **SVG** (current prototype approach): fine to ~a few hundred nodes, easiest to style/interact, but degrades past that.
- **Canvas** (`vis-network` and `react-force-graph`): comfortable into the low thousands.
- **WebGL** (`sigma`, `react-force-graph` 3D): thousands to tens of thousands, with level-of-detail and label culling.

### Interaction model (from the vault's Interface Model)

Build toward, in order:

1. Preview graph above Backlinks (already moved there per [[Session Log]]).
2. Full graph modal with click-to-select nodes (done).
3. **Local graph mode** — subgraph around the active note at depth N (a `get_neighbors` call).
4. **Global graph mode** — whole workspace.
5. **Filters** — by node type, edge type, tag, folder, and provenance. Provenance filter is distinctive to Graphify: let users hide `inferred` edges to see only hard `extracted` truth.
6. **Path finder** — highlight the path between two selected nodes (`find_path`).
7. **Hover/click explanation panel** — show edge type, provenance, confidence, source (`explain_edge`).
8. **Query panel** — run a saved graph query and render the result subgraph.

### Visual encoding conventions

- Node **color** = node type; node **size** = degree/centrality (hubs read bigger).
- Edge **style** = provenance: solid = `extracted`, dashed = `inferred`, accent color = `manual`. This makes the truth model visible at a glance and is worth adopting as a house convention.
- **Communities** = subtle background hulls or shared node tint per cluster.
- Respect the app's theme system (dark default, switchable themes from [[Session Log]]) — drive graph colors from CSS variables, not hardcoded values.

## What's doable in Diamante — prioritized

### Now (browser prototype, React/Vite — no Tauri needed)

- Keep the full graph modal on **`vis-network@9.1.6`** so it stays aligned with Graphify's open-source HTML exporter.
- Keep a deterministic **1,000-node dummy vault fixture** available for graph renderer, filter, and interaction stress testing before real vault/source indexing lands.
- Formalize the **node/edge schema** (with provenance/confidence) as the in-memory model + a `graph.json` snapshot in `.diamante/`. (Backlog: "Node and edge schema for unified graph", "Edge provenance".)
- Add **`graphology`** as the analysis engine: communities, most-connected, shortest path — all runnable in-browser.
- Ship **local vs global graph modes**, **filters** (type/tag/provenance), and **path highlighting** — all pure frontend.
- Add **tree-sitter (WASM)** parsing of any source files the user opens, to start emitting real code edges even before the desktop shell exists.

### Needs the Tauri shell (Phase 1+)

- Real filesystem graph over an actual vault folder + source repos (vs localStorage seed notes).
- **Filesystem watcher → incremental re-index** (Rust file watcher + hash-based diffing).
- Native **tree-sitter** / LSP extraction across a whole repo.
- Move the store from `graph.json` to **SQLite** (`rusqlite`) once size warrants; KuzuDB only if graph queries get complex.
- **Rebuild graph after git sync** (ties into [[GitHub Sync]]).

### The MCP server (parallelizable, biggest agent-workflow payoff)

- Stand up a **TypeScript MCP server** reading the same store, exposing the tool set above over stdio. Doable as soon as the schema + `graph.json` exist — does not require Tauri.
- This is what turns Diamante from "a nice editor" into the Dakila unified system, so it is worth pulling earlier than its Phase 4 slot if agent workflows are a priority.

## Recommended concrete stack

- Extraction: Markdown AST (now) + tree-sitter WASM (soon) + tree-sitter native / LSP (Tauri).
- Store: `graph.json` in `.diamante/` now → SQLite (`sql.js` browser / `rusqlite` native) → KuzuDB only if needed **(verify vs Graphify)**.
- Analysis: `graphology` + its algorithms, shared by UI and MCP server.
- Visualization: `vis-network@9.1.6` for Graphify parity now; WebGL/`sigma` only if scale demands.
- Agent access: MCP server (TS SDK, stdio) exposing search/neighbors/path/explain/impact/communities + note/file resources.

## Open questions

- Graphify's official local artifact is `graphify-out/graph.json`; Diamante should stay compatible with this format while keeping its own richer app model.
- Graphify's documented MCP server exposes `query_graph`, `get_node`, `get_neighbors`, `shortest_path`, `get_community`, `god_nodes`, `graph_stats`, `list_prs`, `get_pr_impact`, and `triage_prs`, plus read-only graph resources. Diamante now implements the core local graph tool subset.
- Electron-vs-Tauri is still open in [[Tauri Desktop Shell]]; note that all of the above works under either shell, so the graph/MCP work is not blocked on that decision.
- How aggressively to compute `inferred` edges in v1 — start with `extracted` only (links, imports, manifests) and add inference later to keep confidence high.

Related: [[Unified Knowledge System]], [[Architecture]], [[Backlinks and Indexing]], [[Product Roadmap]], [[GitHub Sync]], [[Tauri Desktop Shell]], [[Decision Log]]
