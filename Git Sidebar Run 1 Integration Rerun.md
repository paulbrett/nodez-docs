---
id: nodez-git-sidebar-run-1-integration-rerun
title: Git Sidebar Run 1 Integration Rerun
type: roadmap
status: draft
created: 2026-09-11
updated: 2026-09-11
tags:
  - git
  - orchestration
  - execution
---

# Git Sidebar Run 1 Integration Rerun

## Purpose

Scoped rerun of GIT-03 from [[Git Management Sidebar Execution Plan]]. GIT-01 and GIT-02 are treated as
accepted: GIT-01 was merged to `main` by hand, and GIT-02 is merged from its worker branch before launch.
GIT-03 therefore carries no dependency and runs as the single task in this run.

> [!important] Human launch and merge gates
> Importing this plan does not start workers. Starting orchestration is explicit. The worker result requires
> independent review, and merging the accepted branch remains a separate human action. Do not commit, push,
> publish, or delete files as part of this run unless separately authorized.

## Run policy

| Setting | Value |
| --- | --- |
| Provider pool | All available providers |
| Coordinator floor | Architect / `architecture` |
| Builder routing | Lowest capable model satisfying the recorded floor |
| Refuter routing | Prefer a different provider/model from the builder |
| Concurrency | One builder; review runs sequentially |
| Builder permission | Full access, limited to owned paths |
| Refuter permission | Read-only |
| Base branch | `main`; capture the actual HEAD at launch |
| Merge policy | Human-approved worker merge; leave integration uncommitted |
| Source of intent | This note, [[Git Management Sidebar Execution Plan]], and [[Git Management Sidebar Plan]] |

Base revision before the GIT-02 merge was `1573ef72dcf81ef0ed7fe1a601cb43bbf91a6684`. Recheck at launch;
the GIT-02 merge changes it.

Prohibited inherited paths: `src/features/orchestration`, `src-tauri/resources/mcp/bundle.json`. The
generated MCP bundle must not be regenerated or restored as incidental cleanup.

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

## Run 1 — Repository switching and inspection integration

### Goal text

> Wire the read-only Git Management Sidebar inspection slice into the running app. Register the native Git
> inspection commands from the existing `src-tauri/src/git.rs` module, compile and run its tests for the first
> time, and mount the existing `src/features/git` Source Control sidebar as an activity-bar mode. Preserve
> current workspace tabs, editor/chat/orchestration drafts, existing vault sync behavior, and the
> commit-driven repo-index signature. Finish only after independent review and focused plus full validation.

### Task GIT-03 — Run 1 integration seam

| Field | Assignment |
| --- | --- |
| Role | Systems engineer |
| Capability floor | `protocol-reasoning` |
| Permission | Full access |
| Owned paths | `src-tauri/src/lib.rs`, `src/App.tsx`, `src/shared/types.ts`, `src/features/workspace/sourceRoot.ts`, `src/styles.css` |
| Prohibited paths | `src/features/orchestration`, `src-tauri/resources/mcp/bundle.json` |
| Dependencies | None |

Register the native inspection commands and mount Source Control as an activity-bar sidebar mode. Integrate it
without replacing Notes, Code, AI Chat, or Orchestration main panels. Route repository and workspace generation
explicitly. Open selected diffs through the existing editor facilities where compatible and provide honest
metadata views for unsupported files.

Keep the legacy vault sync panel working and visibly separate. Avoid a second source of Git state in
`App.tsx`; adapt old status consumers to the new inspection result when safe, without changing mutation
semantics in this run.

Known starting condition, verified before launch: `src-tauri/src/git.rs` is present on `main` but is not
declared in `lib.rs`, so it is not part of the crate. Its seven `#[test]` functions have never been compiled
or executed. Adding `mod git;` is part of this task and is the first time that module is built. If `git.rs`
fails to compile or its tests fail, that is a real defect to report and fix within the owned integration
surface where possible; do not delete or stub the module to make checks pass, and do not silently narrow its
behavior. If a fix requires editing `git.rs` itself, stop and report it as a scope boundary rather than
editing a path this task does not own.

`src/features/git` is present from the merged GIT-02 branch and is not owned by this task. Consume it through
its exported contract; do not rewrite it. If its props contract does not fit the real workspace state, report
the mismatch rather than reshaping the feature.

Required evidence:

- Repeated Notes/Project switching never crosses roots, branches, counts, diffs, loaders, or errors.
- Source Control remains mounted or restores its state across every main workspace tab.
- Workspace replacement invalidates old requests.
- Existing GitHub sync, editor, AI Chat, and Orchestration interactions still open.
- Dirty status does not change the indexing signature or start the indexer.
- `git.rs` compiles as part of the crate and its module tests execute with recorded results.

Checks:

- `cargo test --manifest-path src-tauri/Cargo.toml --lib git`
- `node --experimental-strip-types --test scripts/git-sidebar.test.mjs`
- `node --experimental-strip-types --test scripts/repository-editor.test.mjs`
- `node --experimental-strip-types --test scripts/orchestration.test.mjs`
- `npm run lint`
- `npm run build`
- `cargo test --manifest-path src-tauri/Cargo.toml --lib`
- `git diff --check`

### Run refuter

Assign a read-only Security reviewer or Systems engineer from an independent provider. Review the actual
combined diff and rerun the focused tests. Attempt to refute repository isolation with delayed-response,
identical-filename, shared-worktree, nested-root, unusual-path, and unsupported-diff cases. Confirm that the
newly compiled `git.rs` tests genuinely executed rather than being filtered out by the test-name argument.

A pass report must name the reviewed commit/base, files inspected, exact commands and results, remaining
manual checks, and any behavior that relies only on simulated fixtures.

### Human acceptance

Manually inspect Notes and Project repositories with distinct branches and identical filenames. Verify the
selected role and root remain visible in the sidebar and diff header. Check keyboard use and narrow layout.
Accept and merge only after the refuter passes.
