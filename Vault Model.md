---
id: nodez-vault-model
title: Vault Model
type: vault
status: active
created: 2026-08-19
updated: 2026-08-19
tags:
  - vault
  - markdown
---

# Vault Model

A vault is a folder containing Markdown notes and attachments.

This mirrors Obsidian's core idea: the app is an interface over regular files, not the owner of the knowledge base.

## File rules

- Notes are `.md` files.
- Attachments can live in `attachments/`.
- App metadata can live in `.nodez/`.
- Obsidian compatibility settings can live in `.obsidian/`.

## Note identity

In the simplest version, note identity is the file path.

Later, Nodez may support stable note IDs in frontmatter:

```yaml
---
id: 2026-08-19-nodez-home
---
```

## Links

Support these first:

- `[[Note Title]]`
- `[[Folder/Note Title]]`
- `[[Note Title#Heading]]`
- `[[Note Title|Alias]]`

Related: [[Architecture]], [[Backlinks and Indexing]]
