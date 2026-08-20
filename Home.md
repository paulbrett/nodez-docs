---
id: diamante-home
title: Diamante Notes
type: docs-index
status: active
created: 2026-08-19
updated: 2026-08-20
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
- [[Graphify Tech Research]]
- [[Repo Indexing]]
- [[GitHub Sync]]
- [[Product Roadmap]]
- [[Backlog]]
- [[Decision Log]]
- [[Session Log]]
- [[Next Steps]]

## Current prototype

The first prototype is a browser-based React app with:

- CodeMirror Markdown editing
- Markdown preview
- local browser storage
- wikilink parsing
- backlinks
- outgoing links
- tags
- small graph view
- GitHub sync workflow UI

## Next milestone

Phase 1 is done: the [[Tauri Desktop Shell]] builds and runs, opening a real folder as a vault, with a file-explorer tree, on-disk rename/delete (with `[[wikilink]]` rewriting), and reconciliation of external edits via a filesystem watcher. Next up is Phase 2 (GitHub sync) and starting Phase 6, indexing an external project/repo alongside the vault. See [[Next Steps]] and [[Repo Indexing]].
