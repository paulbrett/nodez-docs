---
id: nodez-git-management-sidebar-execution-plan
title: Git Management Sidebar Execution Plan
type: roadmap
status: draft
created: 2026-09-10
updated: 2026-09-10
tags:
  - git
  - orchestration
  - execution
  - workspace
---

# Git Management Sidebar Execution Plan

## Purpose

This is the orchestration-ready delivery plan for [[Git Management Sidebar Plan]]. It converts the product design into bounded assignments, dependencies, ownership, review gates, and acceptance evidence.

Use three linked orchestration runs. Do not launch all phases as one run: later mutation and remote work must build on the reviewed contracts from the inspection slice.

> [!important] Human launch and merge gates
> Creating or reviewing a run does not start workers. Starting orchestration is explicit. Each worker result requires independent review, and merging an accepted worker branch remains a separate human action. Do not commit, push, publish, or delete files as part of these runs unless the user separately authorizes that operation.

## Run policy

Use these settings for every run:

| Setting | Value |
| --- | --- |
| Provider pool | All available providers |
| Coordinator floor | Architect / `architecture` |
| Builder routing | Lowest capable model satisfying the recorded floor |
| Refuter routing | Prefer a different provider/model from the builder when ready |
| Concurrency | Two builders; integration and review run sequentially |
| Builder permission | Full access, limited to owned paths |
| Refuter permission | Read-only |
| Base branch | `main`; capture the actual HEAD at launch |
| Merge policy | Human-approved worker merge; leave integration uncommitted |
| Source of intent | This note and [[Git Management Sidebar Plan]] |
| Implementation truth | The checked-out source and tests at the captured base revision |

At plan creation the repository was on `main` at
`c2816a07c7ad6e0106ad62e2b2917871142b9e65`. Recheck this value at launch.

The following inherited local changes were present and belong to existing work:

- `src/features/orchestration/OrchestrationSetup.tsx`
- `src/features/orchestration/OrchestrationWorkerRunner.tsx`
- `src-tauri/resources/mcp/bundle.json`

All Git sidebar workers must treat those paths as prohibited. The coordinator must refresh the dirty-file inventory at launch. The generated MCP bundle must not be regenerated or restored as incidental cleanup.

## Shared invariants

Every assignment includes these requirements:

- Preserve the source-index signature as `branch|HEAD`; dirty-file refresh cannot trigger repo re-indexing.
- Keep Notes and Project repository identity visible and scoped through requests, responses, diffs, errors, and completion messages.
- Git state describes saved disk content. Unsaved editor buffers and unapplied AI proposals remain separate states.
- MCP remains vault-note-only. Git mutation authority is a separate native capability.
- Native commands resolve registered repository contexts; mutation commands do not accept an arbitrary client path as authority.
- Use argument arrays with no shell interpolation. Handle unusual filenames through machine-readable, NUL-delimited Git output.
- Bound process time, output, pagination, and diff/file sizes.
- Fast-forward-only pull, no force push, no hard reset, no implicit repository initialization.
- Switching repositories or workspace tabs cannot retarget an in-flight action or allow a stale response to replace current state.
- Builders do not modify shared integration files unless their assignment explicitly owns them.
- A passing unit test supports the behavior it exercises; it does not certify desktop interaction, remote authentication, or another platform.

## Run 1 — Repository switching and inspection

### Goal text

> Implement the read-only Git Management Sidebar inspection slice from [[Git Management Sidebar Plan]]. Add a persistent Source Control sidebar with explicit Notes/Project identity, isolated repository state, status groups, and bounded diffs. Preserve current workspace tabs, editor/chat/orchestration drafts, existing vault sync behavior, and the commit-driven repo-index signature. Finish only after independent review and focused plus full validation.

### Task GIT-01 — Native Git inspection contract

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `protocol-reasoning` |
| Permission | Full access |
| Owned paths | `src-tauri/src/git.rs` |
| Prohibited paths | `src-tauri/src/lib.rs`, `src/App.tsx`, `src/styles.css`, `src/features/orchestration`, `src-tauri/resources/mcp/bundle.json` |
| Dependencies | None |

Implement a self-contained native Git module for registered Notes and Project contexts. Define repository identity, capabilities, status entries, conflict/staged/unstaged/untracked states, canonical worktree root, Git directory identity, branch/HEAD/upstream metadata, and bounded read-only diff operations.

The module must parse porcelain v2 or another machine-readable NUL-delimited format without losing rename paths or unusual filenames. Add focused Rust unit/integration tests inside the module using temporary repositories. Do not register Tauri commands in `lib.rs`; instead expose the functions and command handlers the integration task can register.

Required evidence:

- Separate Notes and Project contexts cannot be confused.
- Shared worktree detection produces one underlying repository identity.
- Nested-root scope mismatch is reported and mutation capability is false.
- Staged plus unstaged edits to one file remain two distinct states.
- Renames, deletes, conflicts, untracked files, detached/unborn HEAD, unusual filenames, binary diffs, and size bounds are covered.
- Read-only diff helpers disable external diff and text-conversion execution.

Checks:

- `cargo test --manifest-path src-tauri/Cargo.toml --lib git`
- `cargo fmt --manifest-path src-tauri/Cargo.toml -- --check`

### Task GIT-02 — Frontend Source Control state and sidebar

| Field | Assignment |
| --- | --- |
| Role | Implementer |
| Capability floor | `routine-implementation` |
| Permission | Full access |
| Owned paths | `src/features/git`, `scripts/git-sidebar.test.mjs` |
| Prohibited paths | `src/App.tsx`, `src/styles.css`, `src/shared/types.ts`, `src/features/orchestration`, `src-tauri/resources/mcp/bundle.json` |
| Dependencies | None |

Create the feature-local types, typed bridge, repository-scoped reducer/state model, Source Control sidebar, repository selector, status groups, empty states, filters, and read-only diff selection contract. Put feature styling in a stylesheet under `src/features/git` imported by the feature.

The component receives configured Notes/Project roots and integration callbacks as props. It must not read global workspace state implicitly. Retain commit drafts, expanded groups, selected file, history filter, and scroll position independently per context even though commit/history controls are not enabled in Run 1.

Required evidence:

- Notes and Project are labeled by role, display name, abbreviated path, branch, and count.
- Identically named files in both repositories never share state or selected diff.
- Delayed responses from a prior repository or workspace generation are discarded.
- Switching main workspace tabs does not reset Source Control state.
- No-vault, no-project, non-Git, shared-repository, and scope-mismatch states are explicit.
- Keyboard navigation, focus, accessible names, narrow width, loading, empty, error, and retry states are covered.
- The UI never counts unsaved buffers or proposed edits as Git changes.

Checks:

- `node --experimental-strip-types --test scripts/git-sidebar.test.mjs`
- `npm run lint`

### Task GIT-03 — Run 1 integration seam

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `protocol-reasoning` |
| Permission | Full access |
| Owned paths | `src-tauri/src/lib.rs`, `src/App.tsx`, `src/shared/types.ts`, `src/features/workspace/sourceRoot.ts`, `src/styles.css` |
| Prohibited paths | `src/features/orchestration`, `src-tauri/resources/mcp/bundle.json` |
| Dependencies | GIT-01 and GIT-02 accepted |

Register the native inspection commands and mount Source Control as an activity-bar sidebar mode. Integrate it without replacing Notes, Code, AI Chat, or Orchestration main panels. Route repository and workspace generation explicitly. Open selected diffs through the existing editor facilities where compatible and provide honest metadata views for unsupported files.

Keep the legacy vault sync panel working and visibly separate. Avoid a second source of Git state in `App.tsx`; adapt old status consumers to the new inspection result when safe, without changing mutation semantics in this run.

Required evidence:

- Repeated Notes/Project switching never crosses roots, branches, counts, diffs, loaders, or errors.
- Source Control remains mounted or restores its state across every main workspace tab.
- Workspace replacement invalidates old requests.
- Existing GitHub sync, editor, AI Chat, and Orchestration interactions still open.
- Dirty status does not change the indexing signature or start the indexer.

Checks:

- Focused Run 1 Rust and Node tests
- `node --experimental-strip-types --test scripts/repository-editor.test.mjs`
- `node --experimental-strip-types --test scripts/orchestration.test.mjs`
- `npm run lint`
- `npm run build`
- `cargo test --manifest-path src-tauri/Cargo.toml --lib`
- `git diff --check`

### Run 1 refuter

After GIT-03 finishes, assign a read-only Security reviewer or Systems engineer from an independent provider when available. Review the actual combined diff and rerun the focused tests. Attempt to refute repository isolation with delayed-response, identical-filename, shared-worktree, nested-root, unusual-path, and unsupported-diff cases.

A pass report must name the reviewed commit/base, files inspected, exact commands and results, remaining manual checks, and any behavior that relies only on simulated fixtures.

### Run 1 human acceptance

Manually inspect Notes and Project repositories with distinct branches and identical filenames. Verify the selected role and root remain visible in the sidebar and diff header. Check keyboard use and narrow layout. Accept and merge only after the refuter passes.

## Run 2 — Staging and local commits

Start from the reviewed Run 1 integration revision.

### Goal text

> Add safe local Git management to the reviewed Source Control sidebar: file-level stage/unstage, staged-only commit, explicit native Git authority, revision checks, conflict states, and serialization shared with legacy vault sync. Never auto-stage during the new commit flow and never allow repository switching to retarget an action.

### Task GIT-04 — Native staging and commit operations

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `security-review` |
| Permission | Full access |
| Owned paths | `src-tauri/src/git.rs` |
| Dependencies | Run 1 accepted |

Implement stage, unstage, and commit-staged against native-issued repository context IDs. Add expected HEAD/index state validation, literal path handling, conflict guards, scoped authorization, and per-worktree operation locks. Shared repository roles must share the lock.

Commit accepts a nonblank message and commits the existing index only. It must not call `git add -A`. Return structured results tied to the original repository identity.

Test exact index contents, partial staged/unstaged files, stale expectations, shared worktrees, permission revocation, conflicts, concurrent operations, empty index, and failed commits.

### Task GIT-05 — Staging and commit UI

| Field | Assignment |
| --- | --- |
| Role | Implementer |
| Capability floor | `routine-implementation` |
| Permission | Full access |
| Owned paths | `src/features/git`, `scripts/git-sidebar.test.mjs` |
| Dependencies | Run 1 accepted |

Add file/group stage and unstage actions, per-repository commit drafts, staged counts, disabled reasons, pending operation state, structured results, and accessible repository-qualified labels. The action retains the originating repository identity if the user switches during execution.

Do not implement discard, hunk staging, amend, branch switching, stash, reset, or force operations.

### Task GIT-06 — Run 2 integration and legacy lock

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `protocol-reasoning` |
| Permission | Full access |
| Owned paths | `src-tauri/src/lib.rs`, `src/App.tsx`, `src/features/workspace/sourceRoot.ts`, `src/shared/types.ts` |
| Dependencies | GIT-04 and GIT-05 accepted |

Register mutations, enforce the native capability, and route legacy vault sync through the same worktree operation lock before exposing new mutation controls. Preserve the legacy stage-all behavior and label it clearly during migration. Refresh the correct repository after an operation without invalidating unrelated sidebar state or triggering dirty-only source re-indexing.

Refute and manually accept Run 2 using staged-only commit contents, cross-repository switching during an operation, concurrent legacy/new attempts, conflicts, and permission revocation. Run the full validation set from Run 1.

## Run 3 — History and remotes

Start from the reviewed Run 2 integration revision.

### Goal text

> Add repository History, file timeline, Fetch, fast-forward-only Pull, and non-force Push to the reviewed Git sidebar. Preserve unsaved buffers, scope every action and response to its originating repository, distinguish locally known remote state from freshly fetched state, and make divergence and authentication failures recoverable.

### Task GIT-07 — Native history and remote contract

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `security-review` |
| Permission | Full access |
| Owned paths | `src-tauri/src/git.rs` |
| Dependencies | Run 2 accepted |

Add bounded paginated history, commit changed-file queries, commit diffs, rename-aware file history, fetch, fast-forward-only pull, and non-force push. Return last successful fetch and structured upstream/divergence/auth/offline states. Use the shared operation lock for ref-changing work.

Before pull, return affected-path information needed for the integration layer's unsaved-buffer gate. Do not merge on failed fast-forward. Do not add upstream setup, initialization, rebase, force push, reset, stash, or conflict editing.

Use temporary repositories and local bare remotes. Cover pagination, no commits, detached HEAD, renamed files, missing upstream, divergence, offline/failing remote, and shared Git directories.

### Task GIT-08 — History, timeline, and remote UI

| Field | Assignment |
| --- | --- |
| Role | Implementer |
| Capability floor | `routine-implementation` |
| Permission | Full access |
| Owned paths | `src/features/git`, `scripts/git-sidebar.test.mjs` |
| Dependencies | Run 2 accepted |

Add repository History and File timeline views with pagination, commit details, decorations, changed-file selection, and commit diffs. Add Fetch, Pull, and Push with repository-qualified accessible names, progress, failure recovery, last-fetch freshness, and ahead/behind state.

Label orchestration events as Run activity and AI edits as Proposed edits wherever the surfaces meet. Do not combine them with Git commit history.

### Task GIT-09 — Run 3 integration and content refresh

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `protocol-reasoning` |
| Permission | Full access |
| Owned paths | `src-tauri/src/lib.rs`, `src/App.tsx`, `src/features/workspace/sourceRoot.ts`, `src/shared/types.ts`, `src/features/editor/RepositoryWorkspace.tsx` |
| Prohibited paths | `src/features/orchestration`, `src-tauri/resources/mcp/bundle.json` |
| Dependencies | GIT-07 and GIT-08 accepted |

Register history/remotes and connect commit-diff selection to the editor. Before pull changes working files, invoke the existing unsaved-buffer decision path. Reload clean notes/source buffers after success, preserve dirty drafts, surface revision conflicts, and refresh vault graph freshness separately from commit-driven project indexing.

Refute and manually accept Run 3 using a local remote plus two clones. Verify divergence stops, failed fetch is not displayed as current, dirty buffers survive, clean buffers reload, file timelines stay repository-scoped, and no action targets the repository selected after it started. Run all focused and full checks from earlier runs.

## Coordinator integration checklist

Before accepting any run, the coordinator records:

- base revision and inherited dirty files;
- worker provider, model, capability floor, permission, and owned paths;
- actual changed files compared with the assignment;
- exact focused and full checks with exit status;
- refuter identity, reviewed diff, verdict, and remaining risks;
- manual desktop scenarios still unverified;
- whether a human merge decision remains;
- confirmation that no commit, push, publication, deletion, or generated-bundle cleanup occurred.

If a builder needs `App.tsx`, `lib.rs`, shared types, orchestration files, or another worker's owned path, it reports the requested seam and stops. The coordinator handles that seam in the integration task or revises ownership before restarting.

## Stop conditions

Mark the affected task blocked and retain its evidence when:

- the captured base changes underneath an active worker;
- inherited edits overlap an owned file;
- the repository context cannot be resolved safely;
- tests reveal an existing contract conflict requiring a product decision;
- the selected provider cannot meet the capability floor;
- a worker exceeds its path ownership;
- platform-specific behavior cannot be tested on the claimed platform.

A blocked task does not lower its acceptance criteria. The orchestrator may branch or escalate it while preserving scope and lineage.

## Completion definition

The feature is complete only when all three runs are accepted and merged by a human, focused and full checks pass on the integrated revision, and manual Notes/Project switching plus staged commit and local-remote scenarios are recorded.

Phase 4 items from [[Git Management Sidebar Plan]] remain follow-on work: branch management, upstream setup, repository initialization, hunk staging, guided conflict resolution, stash, discard, and explicit worker-worktree browsing.

## Related

- [[Git Management Sidebar Plan]]
- [[Agent Orchestration and Context Discipline]]
- [[AI Workspace UI and UX Plan]]
- [[GitHub Sync]]
- [[Repo Indexing]]
