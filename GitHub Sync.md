---
id: nodez-github-sync
title: GitHub Sync
type: sync
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - github
  - sync
  - git
---

# GitHub Sync

Nodez sync should be GitHub-backed rather than proprietary.

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

Nodez should avoid silently overwriting notes. If a merge conflict happens, the app should show both versions and offer a guided resolution screen.

## Version 1 scope

- connect an existing GitHub repo
- clone or open a vault repo
- pull, commit, push
- show status
- show conflicts

## Implementation status (v1 — 2026-08-22)

Desktop shell (Tauri) now wraps vault git via CLI:

| Command | Role |
| --- | --- |
| `vault_git_status` | branch, upstream, ahead/behind, dirty files, conflicts |
| `vault_git_pull` | `pull --ff-only`, then merge pull (never force) |
| `vault_git_commit` | `git add -A` + commit (optional message) |
| `vault_git_push` | push to upstream |
| `vault_git_sync` | pull → commit → push; **stops on conflicts** |

UI: status-bar GitHub sync button + command palette **GitHub sync** opens a panel with Refresh / Pull / Commit / Push / Sync. Auth is host git (SSH, gh, credential helper) — no embedded PAT store yet.

After a successful pull/sync, notes reload from disk.

Still later: guided side-by-side conflict editor, clone/connect wizard, OAuth.

Related: [[Tauri Desktop Shell]], [[Product Roadmap]]

## UI entry (2026-08-22)

Primary control: **GitHub sync** icon in the **notes list footer** (beside the note count), not the bottom status bar. Command palette **GitHub sync** still opens the same panel.
