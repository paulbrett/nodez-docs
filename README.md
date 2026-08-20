---
id: diamante-docs-readme
title: Diamante Project Docs
type: docs-index
status: active
created: 2026-08-19
updated: 2026-08-20
tags:
  - project
  - docs
  - obsidian
---

# Diamante Project Docs

This folder is the official documentation and Obsidian vault for the Diamante project.

Open this folder in Obsidian:

```text
/Users/paulbrettorozco/Documents/Projects/Diamante
```

The runnable app/code lives separately in:

```text
/Users/paulbrettorozco/Sites/Diamante
```

## Vault Contents

- [[Home]]
- [[Project Overview]]
- [[Architecture]]
- [[Unified Knowledge System]]
- [[Graphify Tech Research]]
- [[Repo Indexing]]
- [[Vault Model]]
- [[Backlinks and Indexing]]
- [[GitHub Sync]]
- [[Tauri Desktop Shell]]
- [[Product Roadmap]]
- [[Backlog]]
- [[Decision Log]]
- [[Session Log]]

## Folder Convention

- `~/Documents/Projects/Diamante` is for planning, architecture, roadmap, decisions, and Obsidian notes.
- `~/Sites/Diamante` is for the React/Vite app source code and runnable development server.
- Keep product decisions in this vault before or alongside implementation changes.

## Metadata Convention

Every Markdown note in this vault should start with YAML frontmatter:

```yaml
---
id: diamante-example
title: Example
type: overview
status: active
created: 2026-08-19
updated: 2026-08-19
tags:
  - example
---
```

Run docs checks from the app folder:

```sh
cd /Users/paulbrettorozco/Sites/Diamante
npm run check
```
