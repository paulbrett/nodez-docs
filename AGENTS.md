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

- Treat **this folder** (`C:\Users\webwi\Documents\Nodez`) as the official **Nodez** documentation / Obsidian vault.
- App/code lives at `C:\Sites\nodez-app` (git `paulbrett/nodez-app`). Landing + OTA: `C:\Sites\nodez` → <https://getnodez.app/>
- Product direction: [[Unified Knowledge System]] — Obsidian-style vault + Graphify-style relationship graph.
- Before product, architecture, roadmap, or sync changes, read the relevant notes here (start: [[Home]], [[Agent and Human Setup]], [[Next Steps]]).
- Prefer vault terminology from [[Project Overview]], [[Architecture]], [[Decision Log]], [[GitHub Sync]], [[Vault Model]], [[Backlog]].
- Dakila is the reference workflow model: GitHub = implementation truth; vault = intent/decisions; graph = cross-component links.
- Keep planning docs here. Keep runnable source in the app repo. Do not reorganize notes unless the user asks.
- Vault meta: `.nodez/` (legacy `.diamante/` may still exist on older vaults).
- MCP: `node C:/Sites/nodez-app/scripts/nodez-mcp.mjs` with `NODEZ_VAULT_DIR` (or legacy `DIAMANTE_VAULT_DIR`) pointing at the target vault.
