---
id: nodez-agents
title: Nodez vault agent instructions
type: agent
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - agents
  - instructions
---

# Nodez vault — agent instructions

- Treat **this folder** as the official **Nodez** documentation / Obsidian vault (`Documents/Nodez` — this machine: `C:\Users\webwi\Documents\Nodez`).
- App/code: `…/Sites/nodez-app` (this machine `C:\Sites\nodez-app`; macOS may use `~/Sites/nodez-app`). Landing + OTA: `…/Sites/nodez` → <https://getnodez.app/>

- Product direction: [[Unified Knowledge System]] — Obsidian-style vault + Graphify-style relationship graph.
- Before product, architecture, roadmap, or sync changes, read the relevant notes here (start: [[Home]], [[Agent and Human Setup]], [[Next Steps]]).
- Prefer vault terminology from [[Project Overview]], [[Architecture]], [[Decision Log]], [[GitHub Sync]], [[Vault Model]], [[Backlog]].
- Dakila is the reference workflow model: GitHub = implementation truth; vault = intent/decisions; graph = cross-component links.
- Keep planning docs here. Keep runnable source in the app repo. Do not reorganize notes unless the user asks.
- Vault meta: `.nodez/` (legacy `.diamante/` may still exist on older vaults).
- MCP: `node C:/Sites/nodez-app/scripts/nodez-mcp.mjs` with `NODEZ_VAULT_DIR` (or legacy `DIAMANTE_VAULT_DIR`) pointing at the target vault.

<!-- nodez-agent-contract:start -->
## Agent contract — Nodez

This file tells AI agents how to work with **this Nodez vault** (notes + graph) through **Nodez MCP**.

## What this is

| Piece | Role |
| --- | --- |
| **This vault** | Markdown notes on disk — intent, decisions, project knowledge |
| **Source root** | Optional attached repo — **read-only** through MCP |
| **Graph** | `.nodez/graph.json` relationships (notes + optional code) |
| **MCP** | Host agent tools (Hermes, Cursor, Codex, Claude Desktop, …) |

Attached source root (read-only via MCP): `C:/Sites/nodez-app`

## Hard rules

- Write **vault notes only** through MCP — never mutate the attached source root via this surface
- Soft delete moves notes to `.nodez/trash/` — not hard delete
- One vault per MCP process (`NODEZ_VAULT_DIR` / legacy `DIAMANTE_VAULT_DIR`)
- Stay inside the vault (path containment); escapes must fail
- After writes, check `graph_stats.stale` and call `rebuild_graph` when other agents/apps need a durable graph

## Recommended workflow

1. **Orient** — `graph_stats` (and `stale` / artifact age)
2. **Find** — `query_graph` / `search_notes` / `search_symbols` / `list_notes`
3. **Read** — `read_note` or `nodez://note/…`; for code use graph `sourcePath` + host tools (not MCP write)
4. **Relate** — `get_node`, `get_neighbors`, `shortest_path`, `explain_edge`, `impact_of`
5. **Act (vault only)** — `create_note` / `write_note` / `rename_note` / `delete_note`
6. **Refresh** — `rebuild_graph` when durable freshness matters

Human setup: Nodez → Setup wizard / Settings → **Copy MCP JSON** (absolute vault + script paths).

## Project notes (edit freely outside the markers)

- Goals:
- Do not:
- Preferred agent habits:

Generated/refreshed by Nodez 0.4.0. Vault: `C:/Users/webwi/Documents/Nodez`.
<!-- nodez-agent-contract:end -->
