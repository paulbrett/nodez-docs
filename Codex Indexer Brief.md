---
id: nodez-codex-indexer-brief
title: Codex Indexer Brief
type: architecture
status: active
created: 2026-08-20
updated: 2026-08-20
tags:
  - repo-indexing
  - graphify
  - agents
---

# Codex Indexer Brief

Implementation brief for Nodez's Graphify-style background repo indexer.

## Implemented

- Vault graph remains available immediately while the repo index runs.
- Tauri maps all first-party files under the source root, excluding generated, artifact, and `node_modules` folders.
- Repo loading starts metadata-only, then a second extraction pass attaches content for extractable first-party files up to 200 KB.
- Function/symbol indexing is optional from the graph view and must not run automatically on repo restore.
- Frontend stages repo graph updates as mapped metadata first, then extraction batches of about 40 files.
- `node_modules` is not indexed.
- Symbols use existing graph schema nodes and IDs shaped as `symbol:<file>#<name>`.
- `defines` and `imports` are extracted; same-file `calls` are inferred.
- Command palette routes `functions`, `function Name`, and `fn Name` to repo graph symbol search.
- MCP exposes `search_symbols` over the saved `.nodez/graph.json` graph artifact.
- The file tree shows Markdown filenames, not note titles.
- The attached repo is vault-scoped and stored in `.nodez/workspace.json`.
- Function indexing runs off the UI path: native walking uses a blocking worker thread and graph construction uses a Web Worker.
- Large graph output uses a manifest/chunk layout: `.nodez/graph.json` references Graphify-shaped chunks in `.nodez/graph/`.

Related: [[Repo Indexing]], [[Repo Indexing Phase 6a]], [[Unified Knowledge System]], [[Decision Log]]
