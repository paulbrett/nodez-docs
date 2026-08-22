---
id: nodez-backlinks-indexing
title: Backlinks and Indexing
type: architecture
status: active
created: 2026-08-19
updated: 2026-08-19
tags:
  - indexing
  - backlinks
  - search
---

# Backlinks and Indexing

Backlinks are computed from `[[wikilinks]]`.

The indexer should scan every Markdown file and produce a lightweight metadata map.

## Index data

- note title
- file path
- outgoing links
- backlinks
- headings
- tags
- unresolved links
- modified time

## Initial approach

For small vaults, rebuild the full index after file changes.

For larger vaults, only re-index changed files and update the backlink graph incrementally.

## Search

Version 1 can use simple in-memory search.

Later options:

- MiniSearch
- FlexSearch
- SQLite FTS
- Tantivy if the Tauri/Rust path becomes central

Related: [[Vault Model]], [[Architecture]]
