---
id: diamante-home
title: Diamante Notes
type: docs-index
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - project
  - local-first
  - markdown
---

# Diamante Notes

Diamante Notes is a local-first Markdown workspace inspired by Obsidian and Graphify.

The project goal is to combine the best part of Obsidian - plain files in a vault - with Graphify-style project relationship graphs, while replacing proprietary sync with a GitHub-backed workflow.

## Start here

- [[Project Overview]]
- [[Architecture]]
- [[Unified Knowledge System]]
- [[Agent and Human Setup]]
- [[Agent Skills and Surfaces]]
- [[Distribution Versioning and Updates]]
- [[Landing Page]]
- [[Graphify Tech Research]]
- [[Repo Indexing]]
- [[Command Palette and Agent Surface]]
- [[GitHub Sync]]
- [[Product Roadmap]]
- [[Backlog]]
- [[Decision Log]]
- [[Session Log]]
- [[Next Steps]]

## Current product

Desktop (Tauri) app with:

- CodeMirror Markdown editing and preview
- Real folder vaults on disk
- Wikilinks, backlinks, tags, note tree
- Merged vault + repo graph (2D/3D engines)
- Commit-driven repo re-index and `.diamante/graph.json` artifact
- Command palette and MCP server for AI agents (graph query + vault reads/writes)
- Vault GitHub sync v1 and opt-in vault `AGENTS.md` agent contract
- Public landing + signed OTA (`diamante-landing` Pages, Settings check for updates)

## Next milestone

Agent/human setup **P0–P6 largely done**. **P7 OTA path live**: public [[Landing Page]] + signed [[Distribution Versioning and Updates]] feed on Pages. See [[Agent and Human Setup]] and [[Next Steps]].

**Next (pick one track):**

1. **Ship polish** — tag GitHub Release, OTA smoke test, icons/Authenticode  
2. **Agent install** — no-Node / bundled MCP  
3. **Knowledge** — Phase 3 attachments + search ranking  
