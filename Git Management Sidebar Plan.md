---
id: nodez-git-management-sidebar-plan
title: Git Management Sidebar Plan
type: roadmap
status: draft
created: 2026-09-10
updated: 2026-09-10
tags:
  - git
  - workspace
  - ui
---

# Git Management Sidebar Plan

## Outcome

Add a VS Code-inspired Source Control sidebar available across Notes, Code, AI Chat, and Orchestration. Users can immediately identify the selected repository and switch between the notes vault and attached code/project repository.

This is a proposed implementation plan. No Git operations or app implementation are authorized by this document alone.

## Current implementation

Source inspection on 2026-09-10 found:

- The notes Git sync panel is rendered in `src/App.tsx`; its bridge is `src/features/workspace/sourceRoot.ts`.
- `src-tauri/src/lib.rs` provides vault status, fetch, pull, commit, push, and sync. The existing commit command runs `git add -A`. It cannot implement a staged-only commit experience unchanged.
- Project Git status exists through `git_source_status`, but the inspected workspace bridge exposes mutation commands only for vault sync.
- The Orchestration sidebar presents proposal Changes and setup Timeline. These are different from saved Git changes and commit history.
- Repository editor access in `src-tauri/src/repository.rs` is window-scoped and separate from source indexing.
- Repo re-indexing depends on `branch|HEAD`. Sidebar dirty-state refresh must preserve that contract.

## Placement and repository identity

Add a Source Control button to the left workspace rail. It switches the left sidebar between Explorer and Source Control without replacing the main workspace tab. Preserve the main editor, chat session, orchestration draft, and unsaved buffers.

The top of Source Control always contains:

1. A two-option selector: **Notes** and **Project**.
2. Selected role and display name, for example **Notes · Nodez** or **Project · nodez-app**.
3. The repository root on a secondary line, with full path available through keyboard focus and a copy-path action.
4. Branch, ahead/behind counts, remote/upstream, and last refresh state.
5. A visible change count on each repository option.

Use labels and distinct note/folder icons; color is supplemental. A folder called Nodez in both locations must remain distinguishable by role and path. Every diff header, commit action, error, and completion message carries the same repository identity.

Remember selection per workspace. First opening follows the active Notes or Code surface; AI Chat and Orchestration reuse the last choice or default to Notes when available. Subsequent main-tab changes never silently retarget Source Control. Switching the Git selector changes Git context only, not the open vault, attached source root, agent execution root, or MCP binding.

Switching retains each repository's commit draft, tree expansion, selected file, history filter, and scroll position. Loaders and stale responses remain scoped to their originating repository.

## Sidebar contents

### Changes

Show collapsible groups in this order:

- Conflicts
- Staged Changes
- Changes
- Untracked Files

Each row shows relative path, explicit status (added, modified, deleted, renamed, or conflicted), and applicable actions. Support tree and flat-list modes, filter by path, and separate staged/unstaged entries for partially staged files.

Selecting a row opens a diff in the main workspace with its repository identity and comparison labeled:

- Staged: HEAD to index.
- Unstaged: index to saved working file.
- Untracked: empty file to saved working file.

Use the existing editor's diff facilities where compatible. Deleted and renamed files remain inspectable. Binary and oversized files receive an honest metadata view with a size-limit explanation. Git reflects saved disk content; unsaved editor buffers and unapplied AI proposals get separate badges and are never counted as committed changes.

### Commit and remote controls

Provide a commit-message field stored per repository and **Commit staged to Notes** or **Commit staged to Project**. Disable commit for an empty index, blank message, unresolved conflicts, or unavailable write capability, with a visible reason.

Stage and unstage individual files or an explicitly selected group. Bulk actions name the repository. Never auto-stage on commit. Show the staged file count and branch beside the commit action.

Expose Fetch, Pull, and Push as separate controls with explicit repository labels in menus and accessible names. Ahead/behind counts describe locally known remote refs and show last successful fetch; failed fetch must not look current.

The new sidebar initially uses fast-forward-only Pull. Divergence stops with a clear explanation; it must not silently inherit the old panel's merge fallback. Push does not force. Missing upstream, detached HEAD, no commits, Git unavailable, offline/auth errors, and busy repositories have distinct states.

Keep existing vault sync available during migration, clearly labeled as the legacy stage-all workflow. Route it through the same operation lock before enabling new Git mutations. Retire or redirect its entry point only after equivalent supported journeys are verified.

### History and file timeline

Use **History** for the selected repository's paginated Git commit list: message, author, date, short hash, and branch decoration. Selecting a commit lists changed files and opens a commit diff.

Offer **File timeline** for the selected file, scoped to that repository, with commit entries and rename-aware history where supported. Empty history is valid for an untracked file or a repository with no commits.

Keep orchestration activity under **Run activity** and pending proposals under **Proposed edits** when integrating the two surfaces. Do not merge setup events, proposals, and commits into an unlabeled timeline.

## Repository boundary cases

- No notes vault: Notes is unavailable with an Open vault action.
- No project attached: Project is unavailable with an Attach project action.
- Folder is not a Git repository: show its identity and an explanatory empty state. Initialization is a later explicit action.
- Both roles resolve to the same worktree: show **Shared repository · Notes + Project**, one underlying state and operation lock, and identical counts. Do not imply independent histories or double-count changes.
- Vault or source folder lies inside a larger repository: show the detected Git root and scope mismatch. In the first release, permit scoped inspection but block mutations until the user explicitly opens/attaches the repository root; never silently commit sibling directories.
- Linked worktrees are distinct working contexts even when they share Git metadata. Coordinate ref-changing operations across their common Git directory.
- Nested repos/submodules are identified as such; recursive management and automatic discovery beyond the two configured roots are deferred.
- Orchestration worker worktrees must not appear as the main Project repository. Later worktree selection must display worker, branch, and root explicitly.

## Native and frontend design

Create `src/features/git/` for the repository selector, sidebar, status state, changes tree, history, and typed bridge. Add a dedicated native `src-tauri/src/git.rs` module and register its commands in `lib.rs`.

Model repository contexts with a stable native-issued ID, window/workspace generation, role, configured root, canonical worktree root, Git directory, and capabilities. The backend resolves IDs against registered workspace roots; mutation commands do not trust arbitrary client-supplied roots.

Introduce bounded commands for context discovery, status, diff, history, stage, unstage, commit-staged, fetch, pull, and push. Enforce Git write capability in native code, separate from indexing and provider runtime permissions. Project read-only mode disables Git mutations. Define vault Git capability explicitly rather than borrowing an AI provider's Full access setting. MCP retains vault-note-only mutation authority.

Parse machine-readable NUL-delimited Git status, preserving index and worktree status, old/new rename paths, and unusual filenames. Use argument arrays, literal path handling, path containment, and no shell interpolation. Diff helpers disable external diff/text conversion execution. Bound output, pagination, process duration, and file sizes.

Serialize mutations per worktree and coordinate shared-ref operations. Bind each action to its original repository and expected state; validate branch/HEAD and relevant index/file revisions before applying. Switching the selector during an operation is allowed: progress remains attached to the original target. Workspace closure invalidates stale requests. Cancellation reports the actual result rather than claiming rollback.

Refresh after saves, Git actions, window focus, and debounced filesystem changes; use modest polling only while visible if watching is unavailable. Discard responses from old workspace generations. Keep Git UI refresh independent of graph re-index triggers.

Before operations that change working files, resolve affected unsaved-buffer conflicts through existing Save/Keep draft/Cancel behavior. Reload clean notes and source buffers after successful changes; preserve dirty drafts and surface revision conflicts. Refresh vault note/graph freshness separately from commit-driven source indexing.

## Delivery sequence

### Phase 1: repository switching and inspection

Ship the persistent sidebar, Notes/Project identity, capability and empty states, status groups, bounded read-only diffs, and per-repository state isolation. Existing sync remains the mutation path.

Acceptance: switching repeatedly between two repos with identically named files never shows the other repo's diff, branch, count, or errors. The sidebar works across all workspace tabs without losing drafts.

### Phase 2: local Git management

Add file-level stage/unstage and staged-only commit, native authorization, operation serialization shared with legacy sync, revision checks, and conflict states.

Acceptance: staged-only commit contains exactly the index snapshot reviewed; unrelated unstaged changes remain untouched. Actions started for Notes cannot affect Project after switching.

### Phase 3: history and remote operations

Add paginated repository history, file timeline, Fetch/Pull/Push, upstream state, and refresh of notes/editor content after working-tree changes.

Acceptance: divergent pull stops visibly, failed remote operations remain recoverable, and commit diffs/history always retain repository identity.

### Phase 4: advanced workflows

Consider branch create/switch, upstream setup, repository initialization, hunk staging, guided conflict resolution, stash, and explicit worker worktree inspection. Discard changes requires a preview and confirmation; force push, hard reset, rebase, and repository deletion are outside the initial scope.

## Validation

Use temporary Git repositories with local remotes for native integration tests. Cover separate/shared repos, nested roots, linked worktrees, staged plus unstaged edits in one file, unborn/detached HEAD, renames/deletions, unusual paths, conflicts, auth/fetch failure, permission revocation, stale snapshots, and concurrent legacy/new operations.

Frontend tests cover repository switching during delayed responses and active operations, draft persistence, correct diff comparison, and unavailable states. Verify no dirty-only change alters the source-index signature.

Run lint, build, focused Git state tests, native Git integration tests, and existing repository editor tests after implementation. Manually verify macOS and Windows paths, keyboard navigation, screen-reader labels, narrow-window drawer behavior, and large-repository responsiveness. No runtime tests were performed for this planning document.

## Related

- [[Git Management Sidebar Execution Plan]]
- [[Unified Knowledge System]]
- [[Agent and Human Setup]]
- [[GitHub Sync]]
- [[AI Workspace UI and UX Plan]]
- [[Agent Orchestration and Context Discipline]]
- [[Repo Indexing]]
