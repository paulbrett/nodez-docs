---
id: nodez-graph-scale
title: Graph Scale
type: architecture
status: active
created: 2026-08-20
updated: 2026-08-22
tags:
  - graph
  - graphify
  - performance
---

# Graph Scale

Nodez must keep the full project graph queryable while drawing only a small graph view.

Core rule: the full graph stays indexed and agent-queryable. When the attached repo index is larger than 1,000 files, the canvas receives a draw graph capped around 1,500 nodes. For 1,000 files or fewer, keep the previous full graph behavior. The status bar should communicate both layers, for example `view 412 / index 81240`.

## Current shipped baseline

- The saved graph remains complete and chunked through `.nodez/graph.json` plus `.nodez/graph/`.
- The graph modal derives a capped draw graph before rendering only when the attached repo index is larger than 1,000 files.
- Small repos keep the previous full canvas behavior, including visible `contains` edges.
- For large repos, `contains` edges are hidden from the canvas draw graph, because folder containment dominates large repo views and adds little to overview navigation.
- For large repos, clicking a drawn node toggles a 1-hop expansion from the full filtered graph, capped around 80 neighbors for that node.
- The status bar reads `view N / index M`, where `view` is the rendered draw graph and `index` is the full merged vault plus repo graph.
- Draw graph is shared by both engines (`canvas2d` and `force3d`). Scaling rules apply before the renderer; do not feed the full 80k index to either WebGL or canvas.

## What works at scale

First layer, then click:

- Overview should show root, top folders, vault hubs, and eventually one `node_modules` supernode if dependency indexing returns.
- Symbols stay hidden unless they are hubs.
- Clicking expands 1 hop from the full index, capped at about 80 children with `+N more` surfaced in the details panel.
- Clicking again folds that node.

Cells and supernodes:

- Far zoom should show one node per community or subsystem, sized by count.
- Clicking a cell should explode it into a focused draw graph.

Do not draw folder-tree edges:

- `contains` is most of a large repo graph.
- Use folder hierarchy for placement, not visible edges.
- Draw relationship edges such as `imports`, `calls`, `links_to`, `depends_on`, `references`, and `defines`.

Layout once, freeze, cache:

- Large repos are mostly trees with sparse cross-links.
- Do not run live force physics over the full graph.
- Prefer deterministic hierarchy or radial placement, short force only on a tiny ego graph after expansion, then freeze.
- Cache future positions in `.nodez/layout.json`.

Worker boundary:

- Full adjacency for large indexes belongs off the UI thread.
- React should hold only the draw graph.
- Indexer batches should update a worker, and the worker should post overview or expansion payloads instead of 80k React state objects.
- Search, path, impact, and agent queries should hit the full graph worker or persisted graph, not the canvas.

Renderer split:

| View | Small graph | Large graph |
| --- | --- | --- |
| Graphify | Canvas/vis-style labeled graph | Overview plus expand-on-click only |
| Stars | Instanced points | GPU point cloud, labels only on focus |
| Cells | Community hulls | Far level-of-detail cells |

If Stars still stutters after instancing, evaluate `cosmos.gl` or `sigma.js` plus `graphology`. Do not add multiple new engines at once.

## Do not

- Cap the indexer to fix drawing.
- Run live physics on 80k nodes on the main thread.
- Render 80k labels, meshes, or React node components.
- Auto-expand `node_modules`.
- Auto-fly the 3D camera.

## Ship order

1. Draw graph is separate from the full graph when the repo index is larger than 1,000 files. Done.
2. Expand-on-click from the full index. Done.
3. Hide `contains` edges in the canvas. Done.
4. Freeze and cache layout in `.nodez/layout.json`. **Done 2026-08-22** (seed on open; save after 2D cool-down / 3D engine stop).
5. Move full adjacency/query work into a graph worker. **Done 2026-08-22** (`src/graphQueryWorker.ts` + `graphWorkerClient`; threshold 800 nodes, main-thread fallback).
6. Make Stars use instanced points instead of one mesh per node.

Short paste prompt for future agents: keep the 80k index queryable, but never feed it directly to the renderer. Build a capped draw graph, hide `contains`, expand 1 hop on click, cache layout, and move large adjacency work off the UI thread.

Related: [[Repo Indexing]], [[Unified Knowledge System]], [[TODO]]
