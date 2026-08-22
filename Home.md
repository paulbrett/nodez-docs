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
- Public landing + OTA updater skeleton (`landing/`, Settings check for updates)

## Next milestone

Agent/human setup **P0–P6 largely done** (MCP loop, vault agent contract, git sync v1, extraction regex). **P7 skeleton landed**: in-repo [[Landing Page]] + [[Distribution Versioning and Updates]] (OTA feed + updater UI). See [[Agent and Human Setup]] and [[Next Steps]].

**Next:** enable GitHub Pages + signing secret, tag `v0.3.0` for a live signed feed; no-Node MCP; Phase 3 attachments/search; icons/Authenticode.
