---
id: nodez-september-10-improvements
title: September 10 Improvements
type: architecture
status: active
created: 2026-09-10
updated: 2026-09-10
tags:
  - orchestration
  - markdown
  - vault
  - acceptance
  - release
---

# September 10 Improvements

This checkpoint connects vault execution plans to orchestration, repairs the
worker lifecycle, improves the Nodez Markdown dialect, and adds a guarded
vault-wide Markdown maintenance workflow. It also prevents sibling agent
worktrees from forcing Vite full reloads.

## Source checkpoint

The app changes are committed on `main` as
`c3c98be24ab9cc29a96e261539cd1ac3e86f6a18` —
`feat: add orchestration plan import and markdown maintenance`.

This is a source and local-install checkpoint for version `0.6.0-rc.1`. It is
not a tagged release, notarized package, published update, or Windows
certification.

## Orchestration

- The Goal step can select an execution-plan note and import one run at a time.
- Imported drafts preserve dependencies, scopes, checks, capability floors,
  permissions, shared invariants, prohibited paths, plan identity, and base
  revision warnings.
- Import never starts workers and rejects duplicate application.
- **Start orchestration** queues drafts and launches only dependency-ready tasks
  within the concurrency cap.
- Active runners remain mounted during connection and execution. Connection
  failure, Stop, cancellation, restart, and removal now have explicit settled
  states.
- Removing a task clears its references from dependent tasks.
- Review begins without a preselected pass verdict. Unstructured real reports
  remain distinct from simulations and receive one corrective parsing retry.
- Refuters run read-only on a stronger declared model when available.

The Git sidebar implementation itself remains governed by
[[Git Management Sidebar Execution Plan]]; importing that plan does not mark its
three delivery runs complete.

## Markdown preview and maintenance

- Preview renders Nodez wikilinks and callouts without rewriting syntax examples
  inside inline or fenced code.
- Internal wikilinks open their note and optional heading.
- **Settings → Vault → Markdown health** and the command palette expose a
  vault-wide scan.
- The staged loader reports scan/apply progress. Review separates safe fixes,
  human review, and manual compatibility findings and shows per-note diffs.
- Automatic repairs are limited to syntax-preserving whitespace cleanup.
- Desktop writes are revision-checked and atomically persisted. Changed files are
  skipped rather than overwritten, and successful batches offer one-step undo.
- A clean result uses a centered compact dialog with deliberate header/stepper
  spacing and no disabled Apply action. The full split workspace appears only
  when findings exist.

The local Nodez Markdown host skill records the supported preview dialect.
PyYAML 6.0.3 is installed for its validation tooling; the application continues
to use its existing JavaScript YAML dependency.

## Development reliability

Vite ignores `.worktrees/**` so config files in isolated orchestration
worktrees do not broadcast a full reload to the primary development app and
erase its in-memory session.

## Validation and installation

Completed on macOS 26.5:

- TypeScript, Markdown lint, and 49 docs-frontmatter checks passed.
- The complete JavaScript suite passed: 264 tests.
- All 89 Rust library tests passed.
- Production frontend builds passed.
- The native ARM64 application bundle built successfully with the explicit
  rustup Cargo and Rust compiler.
- The generated ARM64 executable was installed at
  `/Applications/Nodez.app` as version `0.6.0-rc.1`.
- Installed executable SHA-256:
  `503ff0afcd515632eb05ac5308f9fc20a8cc34f02d8e4caa21781d6270086700`.
  It exactly matched the fresh bundle.

Repository-wide `cargo fmt --check` remains red because provider modules already
contain unrelated formatting drift. The Markdown repair code added in
`src-tauri/src/lib.rs` was aligned with rustfmt without expanding the change
into a broad formatting rewrite.

The docs graph refresh includes the updated vault note layer. Its durable
repository source layer remains at the preceding indexed revision,
`c2816a07c7ad6e0106ad62e2b2917871142b9e65`; Nodez must complete its next normal
source re-index before the graph claims the new `c3c98be` revision.

The local app bundle is unsigned and was not launched as a packaged-app smoke
test. Windows installation, signing/notarization, live multi-provider
orchestration, and N−1 to N upgrade testing remain open release gates.

Related: [[Next Steps]], [[Markdown Health and Repair]],
[[Agent Orchestration and Context Discipline]],
[[Git Management Sidebar Execution Plan]], [[September 9 Improvements]]
