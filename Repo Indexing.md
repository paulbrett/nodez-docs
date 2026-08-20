---
id: diamante-repo-indexing
title: Repo Indexing
type: architecture
status: active
created: 2026-08-20
updated: 2026-08-20
tags:
  - graph
  - graphify
  - dakila
  - roadmap
---

# Repo Indexing

Diamante's vault index (Markdown notes, wikilinks, tags) is only half of the [[Unified Knowledge System]]. The other half is indexing an actual project/repo folder the way Graphify does, so code, docs, and vault notes share one graph. This note makes that concrete: what gets built, in what order, and against which real repo.

## Reference target: the Dakila repo

First (and for now, only) target is the Dakila Overland Controller repo:

```text
/Users/paulbrettorozco/Sites/overland/OverlandLightingControllerV1
```

Chosen over the sibling `dakila-landing` static site in the same `/Sites/overland` folder because it is the substantive engineering repo: ESP32 firmware, a Node.js backend, and a React Native mobile app, already carrying its own `AGENTS.md` and a `docs/` folder — a real multi-language, multi-component codebase worth graphing, and the same repo the [[Unified Knowledge System]]'s Dakila-style workflow was written for.

Shape of the repo (excluding `node_modules`, `build/`, `ios/`, `android/`):

- **Firmware** — a handful of hand-written files: `OverlandLightingControllerV1.ino` plus a few `.cpp`/`.h` protocol/client files (`BmcProtocol`, `BmsClient`, `ControllerSocket`).
- **`backend/`** — Node.js (`server.mjs`, ESM), `routes/`, `services/`, `middleware/`.
- **`mobile/`** — Expo/React Native, ~137 `.ts` + 74 `.tsx` files.
- **`docs/`** — ~20 Markdown docs (`Architecture.md`, `Protocol.md`, `Network.md`, `Roadmap.md`, etc.), plus its own `AGENTS.md`.

That mix is exactly the multi-language case [[Graphify Tech Research]] planned for: Markdown/manifest extraction first, tree-sitter for code second.

## What "indexing a repo" adds beyond the vault

The vault index answers "what links to what note." A repo index needs to answer "what imports/calls/depends-on what," across languages, and both must resolve into the *same* graph so a query can walk from a decision note straight to the firmware file it constrains.

Concretely, Diamante needs a second, read-only **source root** (a plain project folder, not a vault of editable notes) alongside the existing vault root, feeding the same [[Unified Knowledge System]] node/edge schema.

## Phased plan

### Phase 6a - Read-only source root (done)

- `index_source_root` walks an arbitrary folder in one native call and returns file metadata (path, extension, size, modified time) plus git state — no note semantics, no frontmatter parsing, no wikilinks, no source content reads.
- The app remembers a source root alongside the vault path in Tauri app state, so both the vault and `/Sites/overland/OverlandLightingControllerV1` can be open at once.
- Graph nodes: `file` and `folder` for everything under the source root, with `contains` edges. This lets the graph UI show "the Dakila repo exists" and lets an agent enumerate it without reading every file.
- A background indexing notice appears while the folder is loading. For git repos, Diamante polls git status and re-indexes when HEAD or porcelain status changes.
- The merged vault+repo graph is written to `.diamante/graph.json` inside the opened vault, using Graphify-compatible JSON plus Diamante metadata.

### Phase 6b - Cheap extraction (Markdown + manifests) (done)

- The source-root walk includes bounded content only for extractable files: Markdown docs, `AGENTS.md`, and `package.json`.
- Markdown headings become repo `artifact` nodes, with `documents` edges from their source file.
- Markdown links and wikilinks become `references` edges to repo files and matching vault notes when Diamante can resolve them.
- `package.json` manifests become `package` nodes with `depends_on` edges for `dependencies`, `devDependencies`, `peerDependencies`, and `optionalDependencies`.
- Extracted metadata is preserved in the Graphify-compatible `.diamante/graph.json` artifact so agents can trace graph answers back to repo files and headings.

Implemented order:

1. Added bounded extractable content to the Tauri source walk.
2. Extended `buildSourceGraph` so file/folder containment remains the base layer, then overlays doc and dependency edges.
3. Added Rust coverage proving Markdown, `AGENTS.md`, and package manifest content is captured while ordinary source files remain metadata-only.
4. Verified TypeScript, build, docs, and Rust tests.

### Phase 6c - Code symbol extraction (first pass done)

- The source walk now captures bounded content for extractable first-party code files up to 200 KB (`ts`, `tsx`, `js`, `jsx`, `py`, `rs`, `cpp`, `c`, `h`, `hpp`, `ino`, `md`, `json`, `html`), while binaries and oversized files remain metadata-only nodes.
- `src/extractSymbols.ts` emits file-scoped symbol IDs (`symbol:<file>#<name>`) with `sourceLine`, using the existing graph schema instead of a parallel representation.
- `sourceGraph.ts` adds `defines` edges from files to symbols/components, `imports` edges for local imports and package imports, and same-file `calls` edges marked `inferred`.
- The frontend paints the vault graph immediately, then stages repo indexing as map-first followed by extraction batches of about 40 files. Aborted/restarted runs keep the last good graph instead of emptying the repo graph.
- Function indexing is enabled by default, but it starts lazily in the background only after the graph view is opened. Settings can turn it off for metadata-only repo maps.
- `node_modules` is excluded entirely. Repo indexing stays focused on first-party source and root/workspace manifests; dependency graph detail comes from first-party `package.json` files, not vendor folders.
- By default, source walking follows `.gitignore` through the Rust `ignore` walker. Settings now expose an `Ignore .gitignore` toggle for cases where hidden/generated files should still be indexed.
- Large graph artifacts are chunked: `.diamante/graph.json` is a small manifest, while Graphify-shaped node and edge chunks live under `.diamante/graph/`. MCP reconstructs the graph from those chunks for agent queries.
- Large graph rendering follows [[Graph Scale]]: keep the full index queryable, derive a capped draw graph for the canvas, hide noisy `contains` edges, and expand from the full graph on click.

Future hardening can still replace the regex extractor with tree-sitter for richer syntax coverage, but the current pass already produces a complete first-party file graph plus practical symbols, imports, calls, and package dependency nodes from first-party manifests without forcing one huge JSON file.

### Phase 6d - Unified graph + MCP (partially done)

- Merge the source-root graph into the same in-memory graph the vault already builds (`buildKnowledgeGraph` in `src/graph.ts`), so local/global graph modes, filters, path finder, and the Explain panel all work across vault notes and repo files together.
- Extend the MCP server's tool surface (already covering the vault) to also serve `search_nodes`/`get_neighbors`/`find_path`/`impact_of` over the repo side, so an agent working in the Dakila repo can query relationships before reading files, per the [[Unified Knowledge System]] agent model. Current shipped MCP addition: `search_symbols` over the saved `.diamante/graph.json` artifact.
- Rebuild-on-change: reuse the same filesystem watcher pattern already shipped for the vault (see [[Next Steps]]) for the source root too.

## Non-goals for v1

- No write access to the source root. Diamante reads code; it does not edit it.
- No cross-repo indexing yet — one source root (Dakila) proves the model before generalizing to "any number of source roots."
- No full type-checker/LSP-grade call resolution — tree-sitter's approximate `calls` edges are marked `inferred` and are good enough for "what likely touches this," not a source of implementation truth.

Related: [[Unified Knowledge System]], [[Graphify Tech Research]], [[Architecture]], [[Product Roadmap]], [[Next Steps]], [[Decision Log]]
