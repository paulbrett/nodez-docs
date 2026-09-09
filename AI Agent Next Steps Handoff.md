---
id: nodez-ai-agent-next-steps-handoff
title: AI Agent Next Steps Handoff
type: roadmap
status: active
created: 2026-09-09
updated: 2026-09-09
tags:
  - agents
  - handoff
  - acceptance
  - roadmap
---

# Nodez Next Steps Implementation Plan

> For agentic workers: execute one assigned task at a time using the host's
> executing-plans workflow when available. Parallel workers require explicit
> ownership and coordination. This handoff does not itself start any agents.

**Goal:** Finish a verifiable release candidate, then deliver portable attachments
and trustworthy graph freshness without regressing vault ownership or editing.

**Architecture:** Preserve the React/Tauri workspace, window-scoped provider
bridges, explicit repository saves, and vault-only MCP writes. Release readiness,
attachments, and graph work are separate subprojects with separate acceptance
records. Integrate through existing boundaries instead of restructuring App.tsx.

**Tech stack:** React 18, TypeScript, Vite, CodeMirror 6, Tauri 2/Rust, Markdown
vaults, Node MCP, Codex App Server, and ACP/provider adapters. MCP requires Node.js
20+; development/CI currently uses Node 22.

**Specification and context:** [[September 9 Improvements]], [[Unified Knowledge System]],
[[Agent and Human Setup]], [[AI Chat Follow-on Phases]], [[AI Workspace Delivery]],
[[Repo Indexing]], [[GitHub Sync]], and the repo-root AGENTS.md. Source wins for
current behavior; latest accepted vault decisions win for intent.

## 1. Start here: baseline and ownership

App repo on this Mac: `/Users/paulbrettorozco/Sites/nodez-app`.
Docs vault: `/Users/paulbrettorozco/Documents/Projects/Nodez`.
Resolve equivalent paths on other hosts using AGENTS.md; do not copy Mac paths
into committed runtime configuration.

Source checkpoint for this handoff: `main` at
`c9a9b26850134c35443fcb6eb104d81c759a64f6` (2026-09-09), containing the search,
workspace reliability, credential/settings, and app documentation changes.
The implementation was originally uncommitted on top of `f0163b0`; it is now
included in the checkpoint commit. Agents should fetch that revision and still
inventory any newer local changes before delegation.

Pre-existing edits included Cargo manifests/lockfile, credentials, Gemini/Grok
credential calls, readiness, App.tsx, capability copy, and styles. The latest batch
added search and fixes to credentials, reviewed edits, and conversation scope.
Read `git diff` before modifying any of these. Do not reset, stash, overwrite,
stage everything, or claim ownership of all existing changes.

```sh
git status --short
git branch --show-current
git log -1 --oneline
git diff --stat
```

The latest recorded evidence is 200 JavaScript tests, 86 Rust tests, passing
TypeScript/docs checks, successful Apple Silicon app/DMG builds, and a packaged
app launch. These are historical evidence, not substitutes for checking a new
revision. The artifacts are local builds of a dirty tree, not published releases.

### Suggested agent assignments

| Agent | Responsibility | Ownership |
| --- | --- | --- |
| Coordinator | Baseline, integration, release decision, shared UI wiring | App.tsx, shared styles, this handoff, integration commits |
| Provider worker | Protocol fixtures, runtime acceptance, readiness | Chat modules, native provider modules, provider tests |
| Editor worker | Async edit safety and search acceptance | Editor modules, noteSearch/NoteSearchResults, focused tests |
| Release worker | Version comparison, packaging, Windows, update checks | appUpdate/version helpers, release workflow, feed script/tests |
| Attachments worker | Attachment storage and note UX after release work | New attachment modules; shared-file changes coordinated |
| Graph worker | Graph persistence/freshness and citations | Graph modules, graph-lib/MCP tests; shared-file changes coordinated |

Use at most the host's available worker slots. A worker is not alone in the
codebase: preserve others' changes and report any overlap before editing shared
files. App.tsx, src-tauri/src/lib.rs, and shared styles should have one active
writer. Do not have several agents automate the same desktop session.

### Global constraints

- Runnable code stays in the app repo; planning and acceptance notes stay here.
- MCP mutates only its active vault. Never add indexed-source writes to MCP.
- Repository editing is a separate window-scoped capability. Apply changes buffers;
  explicit Save writes disk. Preserve revision checks and undo isolation.
- Git re-index signature stays `branch|HEAD`. Dirty status may warn but cannot
  restart repository indexing or create a continuous function-indexing loop.
- One vault per MCP process. Preserve containment and soft deletion.
- Read-only remains the default provider mode; test actual advertised capabilities.
  Claude has no mid-turn approval channel. Gemini read-only requires Plan support.
- Never put keys, credential files, private transcripts, or signing secrets in logs,
  fixtures, documentation, commits, or handoff messages.
- Use disposable vaults/repos for destructive scenarios. Test upgrades on copies.
- Do not create release tags or dispatch the existing release workflow merely to
  test it: the workflow includes public feed writes and release publication.
- Commit/push/publication actions must follow the authorization present in the
  executing session. This document is an engineering plan, not blanket permission
  to publish, change account access, or replace signing identities.

## 2. Subproject A: finish release readiness

Complete A0 before parallel work. A1/A2/A3/A4 may proceed independently with the
ownership above. A5 packages their integrated result. External blockers should
block the relevant matrix cells, not unrelated work.

### A0 — Preserve and verify the implementation baseline

**Owner:** coordinator.

- [ ] Read repo/vault contracts and September 9 Improvements.
- [ ] Inventory modified and untracked files; distinguish inherited changes from
  new changes. Preserve a reviewable baseline before distributing worktrees.
- [ ] Confirm the actual version in package.json, Cargo.toml, tauri.conf.json,
  and src/shared/version.ts. Current version is `0.6.0-rc.1`.
- [ ] Run the validation commands in section 6 and record failures separately from
  the intended task. Do not silently rewrite inherited work to get a green build.
- [ ] Create `Release Acceptance Matrix.md` in this vault with the schema in A1.

**Exit:** every worker knows its base revision plus any required dirty-tree patch;
there is no ambiguity about what was tested or what remains uncommitted.

### A1 — Complete provider protocol and desktop acceptance

**Files:**

- `src/features/chat/hooks/useAgentSession.ts`
- `src/features/chat/CodexChatPanel.tsx`
- `src/features/chat/chatProvider.ts`, `chatProviderCapabilities.ts`, `codexChat.ts`
- `src-tauri/src/{codex,claude,grok,opencode,gemini,agent_session}.rs`
- `src-tauri/src/{claude_translate,grok_translate,claude_diff}.rs`
- `scripts/codex-protocol-fixture.mjs`, `scripts/fixtures/fake-claude.mjs`
- `scripts/{agent-session,chat-provider,chat-approvals,codex-chat}.test.mjs`
- Proposed new fixture: `scripts/fixtures/fake-acp.mjs`.

**Existing interfaces:** `acceptAgentEvent(identity, event)` scopes events by
workspace/provider/session/thread/turn. `carriedForScope(carried, scope)` restricts
provider handoff text. Native events enter the shared chat reducer through the
existing session hook; do not invent another transport.

- [ ] Use fixtures first. Extend the existing protocol fixtures or add a reusable
  ACP fixture with deterministic scenarios for text, approvals, cancellation,
  errors, late events, model lists, and malformed messages. Fixtures must neither
  execute real commands nor write real source files.
- [ ] For each missing behavior, add a regression before the fix. Example:

```js
assert.equal(acceptAgentEvent(identity, {
  ...event,
  workspace: 'different-vault:repo',
}), false);
assert.equal(carriedForScope({
  scope: 'vault-a:repo', text: 'private context',
}, 'vault-b:repo'), null);
```

- [ ] Verify delayed output after Stop/disconnect/provider switch/vault switch is
  ignored, including workspace A → B → A and a new session in the same workspace.
- [ ] Verify carrying text on a provider switch works only when selected; New
  conversation and explicit saved-session selection discard pending carry text.
- [ ] Keep fixtures and real-provider evidence separate. A fixture proves routing;
  it does not prove current provider authentication or native permission behavior.
- [ ] Complete the live matrix below in temporary workspaces. Use short prompts
  and no real project context. Record observed mode/model and effective behavior.

| Scenario | Procedure | Required outcome |
| --- | --- | --- |
| Connect | Connect with installed CLI and existing auth | Correct provider/model list; no unrelated workspace context |
| Basic prompt | `Reply exactly NODEZ_ACCEPTANCE_OK. Do not use tools.` | Exact response; busy state clears |
| Stop | Start bounded longer output and press Stop during streaming | Returns idle; no continuing output or resurrected turn |
| Reconnect | Disconnect/reconnect; send another minimal prompt | Old process/session cannot contaminate new output |
| Read-only | Ask to modify a designated scratch file | Source unchanged; operation refused or correctly prevented |
| Ask/deny | Request a scratch-file change requiring approval; deny | No write and no misleading success |
| Ask/allow | Repeat with a fresh request; allow once | Exactly requested change; approval is not reused outside its scope |
| Full access | Explicitly select existing supported mode; edit scratch file | Mode is visible; expected scratch write; no implicit mode fallback |
| Images | Send a tiny local fixture image where supported | Correct bounded input; unsupported provider hides/rejects images |
| New/resume | New conversation, saved conversation, provider carry | No accidental carry into a deliberately fresh conversation |
| Failure | Invalid/missing auth, missing CLI, provider exit | Actionable error; no secret exposure; retry remains possible |
| Usage | Provider supplies usage/limit, then omits it | Report valid values; never invent a context limit |

Provider coverage at handoff:

| Provider | Already observed | Still required |
| --- | --- | --- |
| Codex 0.147.0 | Connection and exact response | Modes, approvals, Stop/reconnect, images, session edge cases |
| Claude 2.1.233 | Connection with Haiku and exact response | Read-only/full, Stop/reconnect, images; Ask is not applicable |
| Grok 1.0.13 | Connection, exact response, streaming Stop | Read-only/Ask, permission scoping, reconnect; images not applicable |
| OpenCode 1.18.23 | Connection, model discovery, new conversation/reconnect, exact response | Modes, approvals, Stop, supported-model images |
| Gemini 0.59.0 | Missing-key detection and recovery copy | All authenticated scenarios; user must enter key in Settings → Keys |

Do not request that the user paste the Gemini key into an agent transcript.

**Evidence row format:** date, OS/build, repo revision/patch, provider CLI version,
model, requested/effective mode, scenario, expected, observed, PASS/FAIL/BLOCKED,
redacted log/screenshot path, issue/fix reference. Unsupported features are N/A
with the capability source. Never mark blocked or N/A as passed.

**Exit:** all supported cells pass on a supported release platform, or the release
explicitly excludes a provider/platform until its blocking cells are resolved.

### A2 — Harden readiness and credential error handling

**Files:** `src-tauri/src/provider_readiness.rs`, `agent_credentials.rs`, provider
launcher resolution, `src/App.tsx` readiness UI (coordinator-owned integration).

Current readiness checks CLI version and key presence only. It does not prove a
handshake, usable authentication, model access, or successful prompting. Version
probing currently calls a synchronous unbounded `Command::output()`.

- [ ] Add a hanging version-probe fixture; establish a bounded result instead of
  allowing one CLI to stall readiness indefinitely.
- [ ] Set explicit proposed budgets: 5 seconds per probe, at most 15 seconds per
  provider, and 4 KiB retained output. Verify cancellation cleans up child processes.
- [ ] Reuse actual adapter command resolution, including overrides, Finder/Start
  menu PATH, and Windows `.cmd`/`.exe` behavior. Avoid divergent discovery rules.
- [ ] Surface `not found`, `timed out`, `missing key`, and `probe failed` distinctly;
  leave live authentication/model checks in Connect unless deliberately added.
- [ ] Test malformed and empty stored keys, missing store, read failures, and
  simultaneous set/clear operations. Inspect Windows replacement failure behavior;
  a failed save must not destroy the previously readable key store.
- [ ] Ensure errors never contain key values. Do not present encoded local storage
  as OS-keychain protection, and do not silently migrate credential architecture.

**Exit:** readiness terminates predictably and matches the runtime launcher;
credential failure is recoverable and accurately described.

### A3 — Finish editor and search race coverage

**Files:** `src/features/editor/RepositoryWorkspace.tsx`, `repositoryEditorState.ts`,
`src-tauri/src/repository.rs`, `src/features/notes/{noteSearch.ts,NoteSearchResults.tsx,MarkdownEditor.tsx}`,
`scripts/repository-editor.test.mjs`, `scripts/note-search.test.mjs`.

**Existing interface:** `applyValidatedChanges(docs, changes)` accepts documents
with path/content/revision and validated `{ latest, content }` entries. It throws
if any target buffer changed, closed, or acquired a different revision.

- [ ] Exercise actual UI async boundaries: change or close one of two files while
  validation is pending; deliver a newer proposal; toggle editor write access;
  disconnect/reconnect; switch roots; invoke Apply twice quickly.
- [ ] Assert all-or-nothing buffer application for Apply all, proposal retention
  on failure, no obsolete session/proposal writes, and truthful transcript status.
- [ ] Test Save after an external edit, Reload/cancel, close/cancel, and one-step
  undo of the proposal distinct from earlier typing. Use scratch files only.
- [ ] Test search opening a body match from Preview and from no selected note,
  nested/collapsed folders, duplicate titles, punctuation, Unicode, no results,
  more than 100 results, and source notes changing while a query is active.
- [ ] Verify Down from input, arrows, Enter, focus visibility, and Escape in input.
  Measure a 1,000-note fixture; report measured latency rather than a made-up SLO.
- [ ] Fix concrete failures locally. Do not add fuzzy/vector search or a new editor
  architecture to complete this task.

**Exit:** regression tests and UI evidence cover the actual integration, beyond
unit tests of the pure helpers. No unrelated source file was edited in acceptance.

### A4 — Correct update semantics and release documentation

**Files:** `src/shared/appUpdate.ts`, proposed `src/shared/releaseVersion.ts`,
proposed `scripts/app-update.test.mjs`, `scripts/publish-update-feed.mjs`,
`scripts/publish-update-feed.test.mjs`, `.github/workflows/release.yml`,
`src-tauri/tauri.conf.json`, distribution/README copy where inaccurate.

**Important source finding:** the current app fetches `updates/latest.json` and
opens the download page. The current Tauri config does not configure the native
updater. The feed publisher produces an `installers` manifest. Older notes about
signed, in-app OTA installation are historical and must not be treated as current.

The current comparison splits versions on dots and uses parseInt, so the release
candidate and stable `0.6.0` compare equal. Fix before validating upgrades.

- [ ] Introduce a pure proposed interface:

```ts
export function compareReleaseVersions(left: string, right: string): number;
```

Return negative/zero/positive for valid SemVer; reject malformed versions so the
caller reports an invalid-feed error. Ignore build metadata for precedence and
support the optional leading `v` used by existing feeds. Keep RC precedence.

- [ ] Write these tests before implementation:

```js
assert.ok(compareReleaseVersions('0.6.0-rc.1', '0.6.0') < 0);
assert.ok(compareReleaseVersions('0.6.0-rc.2', '0.6.0-rc.10') < 0);
assert.ok(compareReleaseVersions('0.6.9', '0.6.10') < 0);
assert.equal(compareReleaseVersions('v0.6.0+build.2', '0.6.0+build.9'), 0);
assert.throws(() => compareReleaseVersions('not-a-version', '0.6.0'));
```

- [ ] Integrate comparison into `checkForAppUpdate()`. Exercise HTTP failure,
  malformed JSON/version, equal/older/newer versions, offline behavior, and the
  release-candidate → stable transition through the public check function.
- [ ] Document the current download-based update flow accurately. Do not quietly
  reintroduce native OTA or signing infrastructure; that is a separate decision.
- [ ] Check feed fragments against the actual selected version and required
  installer targets. A partial failed build must not publish an unusable feed.
- [ ] Add Rust tests to CI if absent and make the TypeScript-test Node requirements
  explicit. Inspect workflow publication gates before any workflow dispatch.

**Exit:** current behavior and docs agree; version checks recognize the stable
upgrade; feed tests validate actual artifact URLs/targets without publishing.

### A5 — Certify platform packages and prepare the release handoff

**Owner:** release worker; coordinator integrates A1–A4 first.

- [ ] Build from the integrated reviewed revision. Record exact revision, dirty
  patch if any, toolchain, targets, timestamps, and artifact checksums.
- [ ] macOS: launch package; test onboarding, opening a real scratch vault, search,
  editor saves/conflicts, provider connection, and external MCP launcher resolution.
- [ ] Exercise DMG install/launch on a disposable profile. Verify Apple Silicon;
  build/test Intel or universal2 only if those are claimed supported targets.
- [ ] Windows host: run tests, build NSIS/MSI, install/launch from Start menu, then
  test CLI discovery, credentials, approval dialogs, Stop, explicit file saves,
  non-ASCII/space-containing paths, conflicts, and unsaved-close confirmation.
- [ ] For the current download-based flow, install N−1 on a disposable profile,
  check the N manifest, follow the download flow, install N, and verify copied
  vault contents and intended settings survive. Do not describe this as signed OTA.
- [ ] Check actual signing/notarization status and state it accurately. Discover
  configured secret names only when needed; never print or copy secret values.
- [ ] Produce a release report listing passed/failed/blocked cells, support claims,
  known limitations, artifact paths/checksums, and proposed release notes.
- [ ] Publish only under the release authorization in the executing session. If
  publication is not authorized, stop at the complete, reviewable release report.

**Exit:** no provider or platform is described as accepted based only on compile
success, fixture output, or another platform's smoke result.

## 3. Subproject B: portable vault attachments

This is the next feature after release readiness; it should not block fixing A.
The following is the proposed first-slice contract. Record its short design in
`Vault Attachments.md` before implementation, honoring the executing session's
existing authorization and any material product choices it has already settled.

**Scope:** paste/drop PNG, JPEG, GIF, WebP, and ordinary file attachments into the
active Markdown note. Copy bytes into `attachments/` under the vault. Insert
relative Markdown references. Inline-preview supported raster images; other files
remain links. Exclude attachment management, remote fetching, SVG/HTML rendering,
source-root imports, and automatic orphan deletion from this slice.

**Proposed limits:** 8 files per action, 20 MiB per file, sequential writes. These
are explicit design defaults, not claims about existing behavior.

### B1 — Storage and reference contract

**Create:** `src-tauri/src/attachments.rs`, `src/features/notes/attachments.ts`,
`scripts/attachments.test.mjs`. **Integrate:** `src-tauri/src/lib.rs` and
`src/shared/vault.ts` through the coordinator.

Proposed frontend types:

```ts
export type ImportedAttachment = {
  path: string; // vault-relative: attachments/example.png
  name: string;
  size: number;
  mime: string;
};
export function attachmentMarkdown(
  notePath: string,
  attachment: ImportedAttachment,
): string;
```

- [ ] Use the active window's vault binding; reject absent/stale bindings and paths
  outside the vault. Generate destination names natively, not from trusted client
  directory strings. Reject symlink escapes and use exclusive creation.
- [ ] Sanitize basenames, preserve sensible extensions, and resolve collisions as
  `image.png`, `image-2.png`, etc., without overwriting existing bytes.
- [ ] Validate decoded byte limits natively as well as before sending large data
  over IPC. Restrict preview MIME independently from generic file storage.
- [ ] Compute Markdown destinations relative to the note folder, with encoding
  for spaces, parentheses, hashes, and Unicode. Example acceptance:

```js
assert.equal(attachmentMarkdown('Plans/Today.md', {
  path: 'attachments/photo one.png', name: 'photo one.png',
  size: 12, mime: 'image/png',
}), '![photo one.png](../attachments/photo%20one.png)');
```

- [ ] Test collisions, traversal, symlinks, oversize files, empty/invalid names,
  read-only destination, interrupted writes, and late completion after vault switch.

**Exit:** imported files survive restart and a copied/moved vault, and failed
imports do not overwrite files or insert broken note references.

### B2 — Note editor and preview integration

**Files:** `MarkdownEditor.tsx`, `markdownPreview.ts`, the attachment module,
App.tsx note preview wiring, and a small attachment UI module if needed.

- [ ] Capture paste/drop only in the note editor. Preserve ordinary text paste
  and existing chat-image behavior.
- [ ] Bind each import to vault, note, and originating editor transaction/bookmark.
  If the user switches context before completion, never insert into the new note.
- [ ] Insert successful references in one undoable editor operation. Report each
  failed file clearly and keep successful imports available; do not hard-delete
  files as a compensating action without a defined recovery design.
- [ ] Resolve local preview images through a vault-contained read mechanism with
  bounded loading and URL cleanup. Do not enable unrestricted filesystem serving.
- [ ] Verify nested notes, vault relocation, reopening in another Markdown app,
  keyboard undo, duplicate filenames, partial batch failure, and Git sync of bytes.

**Exit:** a human can paste/drop a file, see a durable portable reference, restart,
and still open it. No attachment content is silently stored outside the vault.

## 4. Subproject C: graph freshness and source navigation

Read [[Repo Indexing]], [[Graph Scale]], and [[GitHub Sync]] first. Current code
already debounces artifact persistence by 800 ms and uses a worker with fallback;
do not implement a second parallel save loop.

### C1 — Reconcile persistence and metadata freshness

**Files:** `src/App.tsx` artifact-save effect, `src/shared/vaultMeta.ts`,
`src/features/graph/graphArtifact.ts`,
`src/features/graph/graphArtifactWorker.ts`, `scripts/graph-lib.mjs`,
`scripts/nodez-mcp.mjs`, and their behavior tests.

- [ ] Trace note writes, external watcher edits, git pull, repo commits, and MCP
  rebuild through in-memory graph and durable artifact persistence.
- [ ] Reproduce two builds completing out of order. Add a per-workspace generation
  guard so obsolete results cannot overwrite newer same-workspace artifacts or
  update another workspace's UI. Retain the current worker/fallback contract.
- [ ] Verify whether MCP rebuild preserves repository provenance/source metadata.
  In the last refresh, graph_stats reported a sourceHead before rebuild and null
  afterward. Treat this as an investigation lead, not proof of lost source nodes.
- [ ] Add a fixture retaining source nodes, edges, provenance, and HEAD while the
  note layer changes; fix confirmed metadata loss at the persistence boundary.
- [ ] Verify post-pull note reload and one settled artifact rebuild. A dirty tree
  must not change the `branch|HEAD` signature or trigger indexing.

**Exit:** live and durable graphs agree after settling; stale completions are
ignored; repository metadata survives a vault-only rebuild.

### C2 — Expose truthful status and usable citations

**Files:** graph status/inspector components, App.tsx integration, chat context
chips, graph query/MCP stats interfaces.

- [ ] Model `building`, `current`, `stale`, and `error` distinctly. Keep uncommitted
  source changes as an independent warning rather than an indexing trigger.
- [ ] Surface last successful persistence time and retry for actual failures;
  do not claim current merely because an in-memory graph exists.
- [ ] Open note/file citations at their recorded line/heading when available;
  preserve explicit-save and unsaved-navigation handling. Missing targets report
  an error and never silently open a similarly named unrelated file.
- [ ] Include provenance and source location in relationship explanations. Do not
  claim inferred edges are extracted facts.
- [ ] Test stale → building → current, worker/save failure, retry, vault switch,
  file rename/deletion, dirty repo without commit, and actual commit re-index.

**Exit:** users and agents can determine freshness and follow a relationship to
its evidence without changing the indexing contract.

## 5. Work order and stop rules

1. A0 coordinator baseline.
2. A1 providers, A2 readiness, A3 editor/search, A4 updates in owned files.
3. Integrate and run checks once; then A5 platform certification.
4. B1/B2 attachments as a separate reviewed feature slice.
5. C1/C2 graph work as another separately reviewed slice. C investigation may run
   during external acceptance blockers if it does not compete for shared files.

Do not expand into new AI providers, autonomous agents, fuzzy/vector search,
collaboration/sync services, an LSP rewrite, or graph decoration during these tasks.
Report blockers concretely: missing credential, absent Windows host, provider
capability mismatch, unavailable signing identity, or required publication decision.
Continue independent work; do not repeatedly run the same blocked check.

## 6. Validation commands and evidence

From the app repo, on the integrated revision:

```sh
npm run check
node --experimental-strip-types --test scripts/*.test.mjs
cargo test --manifest-path src-tauri/Cargo.toml --lib
npm run build
git diff --check
```

Focused commands while changing behavior:

```sh
node --experimental-strip-types --test scripts/chat-provider.test.mjs scripts/agent-session.test.mjs scripts/chat-approvals.test.mjs
node --experimental-strip-types --test scripts/repository-editor.test.mjs scripts/note-search.test.mjs
node --test scripts/nodez-mcp-write.test.mjs scripts/graph-lib.test.mjs
```

Packaging after relevant code is stable:

```sh
npm run tauri -- build --bundles app,dmg
```

Windows, on the Windows host:

```powershell
npm run tauri -- build --bundles nsis,msi
```

Validate current tool/config requirements before adding cross-architecture flags.
`src-tauri/resources/mcp/` is generated: edit `scripts/prepare-mcp-bundle.mjs`, not
its output. Avoid committing timestamp-only bundle.json churn from local builds.

Keep full redacted logs as artifacts; report counts and failures in the handoff.
Do not repeat broad tests after a passing run unless subsequent changes justify it.
After docs writes, rebuild the docs-vault graph and verify `graph_stats.stale=false`.
Do not claim this also re-indexes changed source code: source indexing is commit-driven.

## 7. Required report from each agent

```text
Task ID and owner:
Base revision and inherited dirty-tree changes:
Outcome (complete / partial / blocked):
Files changed:
Behavior before → after:
Tests and exact results:
Live scenarios verified (platform/provider/model/mode):
Artifacts and evidence paths:
Remaining failures or blockers:
Changes needed in shared files:
Commit(s), if authorized and created:
Next recommended task:
```

A passing unit test is not a packaged-app acceptance result. A build is not an
installation test. A missing credential is not a provider pass. A ready artifact
is not a published release.

## 8. Copyable coordinator prompt

> Read the repo AGENTS.md and docs-vault AI Agent Next Steps Handoff.md, plus
> September 9 Improvements.md. Execute A0 first, preserving all inherited dirty
> changes. Then carry out the assigned A tasks with explicit file ownership and
> concise progress reports. Use fixtures before paid prompts, disposable vaults
> for acceptance, and fresh tests for fixes. Do not publish or dispatch the release
> workflow merely to test it. Report blocked provider/platform cells honestly.
> Update the acceptance matrix and docs graph. Deliver a reviewable release report
> before starting the separately scoped attachment and graph features.
