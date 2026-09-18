---
id: nodez-workspace-improvements-plan
title: Workspace Improvements Plan
type: roadmap
status: proposed
created: 2026-09-16
updated: 2026-09-16
tags:
  - chat
  - attachments
  - graph
  - editor
  - search
---

# Workspace Improvements Plan

> [!summary] Outcome
> Deliver four reliability-oriented slices without expanding provider scope or weakening explicit save, workspace, and vault boundaries.

This plan expands the current delivery order in [[Next Steps]] for:

1. Chat recovery and polish
2. Portable vault attachments
3. Graph trust and source navigation
4. Editor and search hardening

Release acceptance remains a separate gate in [[AI Agent Next Steps Handoff]]. This plan does not authorize publication.

## Guardrails

- Preserve MCP vault-only writes, repository read-only defaults, and explicit source saves.
- Do not restructure [[App]] or introduce a new global state layer while validating behavior.
- Inventory the existing dirty worktree before each slice. Keep inherited changes distinct from the slice's changes.
- Use disposable vaults and repositories for conflict, recovery, and destructive-path tests.
- Record passed, failed, and blocked acceptance cells separately. A build or fixture is not desktop acceptance.

## Sequence

### Phase 0 — baseline and test map

**Purpose:** make the implementation state reviewable before adding new behavior.

- Inventory current modified and untracked files, focused tests, and any overlap with the four slices.
- Map each acceptance case below to an existing test or a new focused regression.
- Assign one integration owner for shared chat UI, app wiring, and styles.
- Establish scratch vault and repository fixtures for attachments, graph races, and editor conflicts.

**Exit:** every planned acceptance case has an owner, test location, and safe fixture; existing changes are preserved and attributed.

### Phase 1 — chat recovery and polish

**Priority:** first. This protects in-progress user work and stabilizes the shared workspace surface before other UI changes.

**Scope:**

- Model connecting, streaming, stopping, failed, and complete as independent visible states.
- Preserve draft text and attachments through failed sends, disconnects, and reconnects without an automatic duplicate send.
- Add a compact per-turn record of effective provider, model, effort, mode, and reported usage where available.
- Finish keyboard navigation, focus restoration, screen-reader announcements, reduced-motion behavior, and narrow-window layouts.
- Keep Stop reachable and show stopped only after the runtime acknowledges it.

**Likely areas:** chat reducer/session hook, conversation persistence, composer and panel components, activity/usage components, focused chat fixtures.

**Acceptance:**

- A failed send and a reconnect retain the exact draft and attachments.
- Stop changes state only after acknowledgement; late events cannot resurrect an old turn.
- Unsupported provider metadata remains absent rather than displaying fabricated values.
- Escape closes menus and restores focus; the narrow layout remains usable with keyboard only.
- Existing retry, transcript search, branching, context chips, copy actions, and proposal links remain covered.

**Exit:** focused fixture tests and a desktop accessibility/compact-layout smoke pass both succeed.

### Phase 2 — portable vault attachments

**Priority:** second. Start design only after Phase 1 begins; begin implementation after its exit gate.

**First-slice contract:**

- Accept pasted or dropped PNG, JPEG, GIF, WebP, and ordinary file attachments.
- Copy bytes beneath the active vault in a dedicated attachments location with collision-safe names.
- Insert portable relative Markdown references into the active note.
- Preview only supported raster images; ordinary files remain links.
- Exclude remote fetching, SVG/HTML rendering, source-root imports, automatic orphan deletion, and broad attachment management.

**Design and implementation tasks:**

1. Specify path containment, naming, overwrite avoidance, size/type limits, atomic copy behavior, and failure cleanup.
2. Add desktop commands for bounded, revision-safe attachment writes.
3. Add note-editor paste/drop affordances and insertion behavior.
4. Render safe image previews and clear missing-file fallbacks.
5. Test reopening and moving notes/vaults with relative references.

**Acceptance:**

- A scratch vault can paste and drop every supported type, reopen the note, and retain a portable reference.
- Duplicate filenames never overwrite an existing attachment.
- Failed copies leave no partial reference or orphan byte file.
- An attachment can never escape the active vault or write to the indexed source root.
- Unsupported or missing files show an actionable result without arbitrary local-file serving.

**Exit:** fixture coverage plus a macOS and Windows smoke scenario using a disposable vault.

### Phase 3 — graph trust and source navigation

**Priority:** third. It follows the attachment contract so graph and citation behavior can account for vault-relative note content consistently.

**Scope:**

- Surface graph states as building, current, stale, and error with truthful reasons.
- Version or cancel asynchronous builds so obsolete work cannot persist over newer vault or source state.
- Preserve repository provenance when a vault-only graph rebuild refreshes the note layer.
- Open citations at the recorded note heading, source file, or source line.
- Show relationship evidence: type, provenance, confidence, and target availability.

**Likely areas:** graph worker/client, graph artifact persistence, MCP graph helpers, graph UI/citation navigation, source preview/editor routing.

**Acceptance:**

- A controlled race proves the newest build wins and an obsolete result cannot overwrite its artifact.
- Vault-only rebuild retains source-layer provenance and never writes the source root.
- Current/stale/error UI matches the artifact metadata and reports the reason.
- A citation opens the intended note heading or source line; missing targets offer a precise recovery message.
- Existing MCP graph queries continue to merge live vault notes while stale.

**Exit:** race and provenance tests pass, followed by a large-vault performance measurement and desktop citation-navigation smoke test.

### Phase 4 — editor and search hardening

**Priority:** fourth. This consolidates the explicit-save workflow after the shared note and graph contracts are stable.

**Scope:**

- Exercise actual UI boundaries: external edits, close during proposal validation, double Apply, root/vault switches, stale proposals, save conflicts, and unsaved-close recovery.
- Verify all-or-nothing multi-buffer proposal application with accurate retained proposals on failure.
- Cover ranked search from preview and no selected note, duplicate titles, Unicode/punctuation, nested folders, more than 100 results, and active index changes.
- Add a compact desktop smoke suite for search navigation, explicit Save, reload/conflict recovery, and proposal undo.

**Likely areas:** repository workspace/state, native repository bridge, note search/results, Markdown editor, focused editor and search tests.

**Acceptance:**

- Apply all cannot partially update buffers when one target changes or closes.
- Save detects external changes; Reload, Cancel, and one-step proposal Undo remain distinct.
- Stale provider/session output cannot apply an obsolete proposal after a reconnect or root switch.
- Search keyboard navigation works from the input, preserves focus visibility, and opens the correct note/body match.
- A 1,000-note fixture records real latency; no performance target is claimed without measurement.

**Exit:** UI-boundary regressions and the desktop smoke suite pass on both a normal vault and a conflict fixture.

## Cross-slice verification

After every slice:

```sh
npm run check
node --experimental-strip-types --test scripts/*.test.mjs
cargo test --manifest-path src-tauri/Cargo.toml --lib
npm run build
git diff --check
```

Run the broad suite only after focused checks pass. Record the exact revision and local patch state, desktop platform, fixture paths, and any blocked provider/platform cases.

## Delivery checkpoints

1. **Chat reliable** — recovery and accessibility acceptance complete.
2. **Attachments portable** — storage contract, safety boundaries, and two-platform scratch-vault proof complete.
3. **Graph trustworthy** — race safety, provenance preservation, and citation navigation accepted.
4. **Editing dependable** — proposal/save/search boundary regressions and desktop smoke suite accepted.

## Deferred

- New provider adapters, speculative cost estimates, and automatic worker launching.
- Attachment management UI, remote attachment fetches, SVG/HTML preview, and automatic orphan deletion.
- Fuzzy/vector search and a new editor architecture.
- Graph-schema changes not needed for freshness, provenance, or citation targets.
- Large-scale `App.tsx` restructuring before these acceptance gates.

## Related

- [[Next Steps]]
- [[AI Chat Controls and Live Usage Plan]]
- [[AI Agent Next Steps Handoff]]
- [[September 9 Improvements]]
- [[Agent Orchestration and Context Discipline]]
