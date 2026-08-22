---
id: diamante-docs-readme
title: Diamante Project Docs
type: docs-index
status: active
created: 2026-08-19
updated: 2026-08-21
tags:
  - project
  - docs
  - obsidian
---

# Diamante Project Docs

This folder is the official documentation and Obsidian vault for the Diamante project.

Open this folder in Obsidian (this machine):

```text
C:\Users\webwi\Documents\Diamante
```

Historical / other-machine path:

```text
/Users/paulbrettorozco/Documents/Projects/Diamante
```

The runnable app/code lives separately:

```text
C:\Sites\diamante
```

(or `~/Sites/Diamante` / `/Users/paulbrettorozco/Sites/Diamante` on other machines)

## Vault Contents

- [[Home]]
- [[Project Overview]]
- [[Architecture]]
- [[Unified Knowledge System]]
- [[Agent and Human Setup]]
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
cd C:/Sites/diamante
npm run check
```
