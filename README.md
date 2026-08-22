---
id: nodez-docs-readme
title: Nodez Project Docs
type: docs-index
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - project
  - docs
  - obsidian
---

# Nodez Project Docs

This folder is the official documentation and Obsidian vault for the Nodez project.

Open this folder in Obsidian (this machine):

```text
C:\Users\webwi\Documents\Nodez
```

Historical / other-machine path:

```text
C:\\Users\\webwi\\Documents\\Nodez
```

The runnable app/code lives separately:

```text
C:\Sites\nodez-app
```

(or `C:\\Sites\\nodez-app` / `C:\\Sites\\nodez-app` on other machines)

## Vault Contents

- [[Home]]
- [[Project Overview]]
- [[Architecture]]
- [[Unified Knowledge System]]
- [[Agent and Human Setup]], [[Agent Skills and Surfaces]]
- [[Distribution Versioning and Updates]]
- [[Landing Page]]
- [[Command Palette and Agent Surface]]
- [[Graphify Tech Research]]
- [[Repo Indexing]]
- [[Vault Model]]
- [[Backlinks and Indexing]]
- [[GitHub Sync]]
- [[Tauri Desktop Shell]]
- [[Product Roadmap]]
- [[Next Steps]]
- [[Backlog]]
- [[TODO]]
- [[Decision Log]]
- [[Session Log]]

## Folder Convention

- Docs/Obsidian vault: planning, architecture, roadmap, decisions
- App/code folder: React/Vite/Tauri source and runnable development server
- Keep product decisions in this vault before or alongside implementation changes

## Metadata Convention

Every Markdown note in this vault should start with YAML frontmatter:

```yaml
---
id: nodez-example
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
cd C:/Sites/nodez-app
npm run check
```
