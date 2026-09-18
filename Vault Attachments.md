---
id: nodez-vault-attachments
title: Vault Attachments
type: architecture
status: active
created: 2026-09-16
updated: 2026-09-16
tags:
  - vault
  - attachments
---

# Vault Attachments

Nodez stores user-selected, pasted, and dropped attachment bytes under the active vault's `attachments/` directory. Notes contain portable relative Markdown references, so nested notes link with `../attachments/…` and work after the vault is copied or reopened elsewhere.

## Current scope

- The Markdown editor accepts file paste and drop (up to eight files per action).
- The toolbar picker remains available.
- PNG, JPEG, and WebP images are resized before storage when their largest dimension exceeds 2,048 px. GIFs remain unchanged to preserve animation.
- Images render through the contained native preview bridge; other files remain ordinary Markdown links.
- Imports are bounded to 25 MB of decoded input, named collision-safely, written atomically, and limited to the active vault's `attachments/` folder.

## Safety

Imports are bound to the active vault and note. If the note or vault changes while an asynchronous import is running, no Markdown is inserted into the new context. Preview resolution accepts only normalized paths that resolve back into the root `attachments/` directory.

Out of scope: remote fetches, SVG/HTML rendering, attachment management, and automatic orphan deletion.
