---
id: nodez-tauri-desktop-shell
title: Tauri Desktop Shell
type: architecture
status: active
created: 2026-08-19
updated: 2026-08-19
tags:
  - tauri
  - desktop
---

# Tauri Desktop Shell

Tauri is the preferred desktop direction for Nodez.

It gives the React frontend native capabilities without shipping a full Chromium runtime in the same way Electron does.

## Why Tauri

- smaller app bundle
- native filesystem access through commands
- Rust backend for git and file watching
- good fit for a local-first desktop app

## What Tauri would provide

- open folder dialog
- read and write Markdown files
- watch vault changes
- run git operations
- store app settings
- package macOS, Windows, and Linux builds

## Open question

Electron may be easier if we later want closer compatibility with the Obsidian plugin ecosystem.

Decision: see [[Decision Log]]
