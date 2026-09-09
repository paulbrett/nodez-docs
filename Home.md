---
id: nodez-home
title: Nodez
type: docs-index
status: active
created: 2026-08-19
updated: 2026-09-09
tags:
  - project
  - local-first
  - markdown
---

# Nodez

Nodez is a local-first Markdown workspace inspired by Obsidian and Graphify.

The project goal is to combine the best part of Obsidian - plain files in a vault - with Graphify-style project relationship graphs, while replacing proprietary sync with a GitHub-backed workflow.

## Start here

- [[AI Agent Next Steps Handoff]] — task ownership, acceptance, and execution order
- [[September 9 Improvements]] — implementation and validation evidence

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
- Commit-driven repo re-index and `.nodez/graph.json` artifact
- Command palette and MCP server for AI agents (graph query + vault reads/writes)
- Vault GitHub sync v1 and opt-in vault `AGENTS.md` agent contract
- Ranked note search with snippets, highlighting, and keyboard navigation
- Repository editor and five AI provider adapters with scoped conversations
- Public landing and download-based update checks (native OTA is not currently configured)

## Next milestone

Finish provider/platform acceptance for the `0.6.0-rc.1` workspace. Basic prompts
passed for Codex, Claude, Grok, and OpenCode; Gemini requires a user-supplied key.
Windows installation and remaining permission scenarios are separate gates.

[[AI Agent Next Steps Handoff]] defines the order: preserve the reviewed baseline,
finish provider and editor checks, fix release-version comparison, certify the
packages, then implement attachments and graph freshness/navigation improvements.
Node.js 20+ remains required for MCP. No new release has been published.
