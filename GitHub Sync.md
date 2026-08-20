---
id: diamante-github-sync
title: GitHub Sync
type: sync
status: active
created: 2026-08-19
updated: 2026-08-19
tags:
  - github
  - sync
  - git
---

# GitHub Sync

Diamante sync should be GitHub-backed rather than proprietary.

The vault itself is a git repository. Sync is a careful wrapper around git commands.

## Basic sync flow

1. Check local changes.
2. Pull from the remote branch.
3. Detect conflicts.
4. Stage changed vault files.
5. Commit with a generated message.
6. Push to GitHub.

## Authentication options

- GitHub CLI auth
- SSH key
- personal access token
- OAuth later

## Conflict policy

Conflicts should be explicit and visible.

Diamante should avoid silently overwriting notes. If a merge conflict happens, the app should show both versions and offer a guided resolution screen.

## Version 1 scope

- connect an existing GitHub repo
- clone or open a vault repo
- pull, commit, push
- show status
- show conflicts

Related: [[Tauri Desktop Shell]], [[Product Roadmap]]
