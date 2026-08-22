---
id: diamante-home
title: Diamante Notes
type: docs-index
status: active
created: 2026-08-19
updated: 2026-08-21
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
- Command palette and MCP server for AI agents (graph query + vault writes)

## Next milestone

Make Diamante **simple for humans to set up** and **complete for agents to use**: note read tools, graph freshness after agent writes, first-run wizard with one-click MCP export, and a proper app `AGENTS.md` contract — **P0–P3 done**. See [[Agent and Human Setup]] and [[Next Steps]]. Next optional depth: P4 impact/explain tools, P5 Dakila extraction, P6 git sync, P7 distribution.

Then: deeper code extraction (Dakila), real GitHub sync with rebuild-after-pull, **app versioning + OTA**, a **public landing page**, and distribution polish (no-Node MCP).
