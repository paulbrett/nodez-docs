---
id: nodez-code-editor-ai-chat-plan
title: Code Editor and AI Chat Plan
type: roadmap
status: active
created: 2026-09-07
updated: 2026-09-09
tags:
  - editor
  - ai
  - planning
---

# Code Editor and AI Chat Plan

## Delivery status — 2026-09-08

The Codex-first vertical slice is implemented on `feat/codex-editor-chat` for
0.6.0-rc.1. It includes the native App Server bridge, persistent resizable chat,
repository file workspace, reviewed single-file edits, bounded note/file/graph
context, image submission, conversation controls, editor preferences, and
workspace isolation. The detailed shipped behavior and validation record live in
[[Codex Chat Implementation]].

Remaining work is release validation: final live desktop acceptance, Windows
smoke testing, installer/package builds, docs graph refresh, and release
publishing. Multi-file edit transactions and richer language services remain
follow-on work. Claude Code, Grok, OpenCode, and Gemini CLI now share the provider
surface.

## Request and status

On 2026-09-07 the user requested: “update memory get nodez vault and docs --plan for code editor with AI chat”.

Implementation authorized on 2026-09-07 after the user accepted the Codex-first runtime integration recommendation (“lets do it”). Build an optional Codex editor chat using App Server, with commands, file changes, diffs, and explicit runtime approvals. This supersedes the earlier planning-only scope and the no-terminal MVP constraint below for this integration. External agent workflows remain supported. The Claude adapter, Grok ACP adapter, and OpenCode ACP adapter shipped on the feature branch on 2026-09-08. Gemini CLI ACP was added on 2026-09-09. See [[Claude Code Provider]], [[Grok Provider]], [[OpenCode Provider]], and [[Gemini CLI Provider]].

## Durable project context

- App: ~/Sites/nodez-app.
- Official docs vault on this Mac: ~/Documents/Projects/Nodez. ~/Documents/Nodez does not exist here.
- Nodez docs MCP is bound to this vault. Other vault servers are separate workspaces.
- Vault notes hold intent and decisions; git/source holds implementation; graph holds derived relationships.
- Prior memory observation 225 confirms CodeMirror 6, Prettier, lightweight diagnostics, and deferred Monaco.

## Verified baseline

- `src/features/editor/components/` supplies CodeMirror editing, language extensions, formatting, and lint support.
- src/App.tsx openCodeSurface/commitCodeDraft edits the active note body or a single fenced code block. It is not a repository file editor.
- `src/features/chat/AskAssistantPanel.tsx` offers single-request note/selection assistance, model settings, cancellation, copy, and insert into note. It does not maintain a conversational transcript or invoke MCP tools.
- Existing integration points: `src/features/chat/askAssistant.ts`, `src/features/chat/assistantSettings.ts`, `src/shared/commands.ts`, `src/styles.css`, and native commands in `src-tauri/src/lib.rs`.
- The docs graph reported stale=false before this update; its sourceHead is an indexed snapshot, not proof of current source freshness.

## Proposed experience

A file tree on the left, tabbed code editor in the center, and a resizable AI chat panel on the right. Keep notes and graph navigation available. Selecting a symbol or file from the graph opens the relevant editor location.

User flow: open workspace → open code file → select code → ask/explain/refactor → review proposed diff → apply to buffer → save explicitly.

Chat supports follow-up questions, streaming when the selected endpoint supports it, Stop, retry, and clear conversation. Show the actual context included with each request. Model-generated code is a proposal until the user applies it.

## Scope and boundaries

1. Reuse the existing CodeMirror and formatting stack. Do not replace the editor engine for this milestone.
2. Ship note-code chat first, then explicit repository editing as a separate phase.
3. Attached source remains read-only for indexing and MCP. A future writable editor root is a separate native capability enabled by the user; attaching a repo alone never grants writes.
4. Codex may run commands and propose file changes through its runtime. Start read-only with on-request approvals; show approval details and diffs. No automatic commits/pushes or unattended background sessions.
5. Keep branch|HEAD re-indexing and the once-per-index function scan. Unsaved buffers can be explicit chat context without triggering durable indexing.
6. Model/provider choice remains configurable through the existing adapter. Validate provider behavior during implementation rather than assuming streaming or tool support.

## Delivery sequence

### Phase 1 — Conversational AI for existing code mode — **implemented**

Extend the current Ask panel with typed messages, pending/error/cancelled states, and a transcript scoped to the active workspace. Start with session-only history; durable history can be added after its storage and deletion behavior are specified.

Include the current code buffer, selected range, language, and explicitly selected notes. Add Explain, Find issue, Refactor, and Write tests prompts. Retain note assistance.

Acceptance: follow-ups include bounded prior turns; context preview matches the request; cancellation and late responses cannot overwrite a newer request or leak into another vault; errors retain the user draft.

### Phase 2 — Repository file workspace — **implemented (single workspace)**

Introduce separate document identities for vault notes and source files. Add source tree, tabs, language detection, dirty state, explicit save, close-with-unsaved handling, and external-change detection.

Open source tabs and the active tab persist per repository and restore only while
their paths remain in the current index. Only paths are stored; dirty buffer
contents are never serialized. Notes keep a bounded, vault-scoped recent list in
the explorer and discard missing or renamed paths during restoration.

Native reads/writes must canonicalize root and file paths, reject symlink/path escapes, bound file sizes, and handle unsupported/binary files. Saves compare the current disk revision with the revision loaded, then write atomically; a mismatch opens a conflict choice and never silently overwrites external edits.

Acceptance: typing and saving affect only the selected editor file; note frontmatter/fences still round-trip; read-only attached repos remain read-only until editor access is enabled.

### Phase 3 — Review and apply AI edits — **implemented for change sets**

Generate a proposal tied to document ID, base content hash, and selected range. Render before/after diff with Apply and Discard. Apply is one undoable buffer transaction, followed by ordinary explicit save.

Acceptance: a stale proposal cannot modify a changed document or a different tab; Discard leaves text unchanged; undo restores pre-apply text. Begin with one file per proposal.

The follow-on reviewed change-set slice now normalizes multi-file provider edits,
shows additions/deletions per file, supports per-file Apply/Discard and Apply all,
and keeps explicit Save as the disk-write boundary. Workspace, turn, authorized
root, and base-revision guards reject stale or misplaced changes.

### Phase 4 — Graph and vault context — **implemented first pass**

Add explicit context chips for selected code, current file, chosen notes, and bounded graph neighbors. Resolve graph paths against the active root; load current file content for citations and label indexed relationship freshness separately.

Acceptance: citations open the right file/note; inferred relationships are distinguishable; missing files and stale graph references produce clear UI; switching vault clears incompatible context and pending requests.

### Phase 5 — Persistence, polish, and validation — **partially implemented**

Specify opt-in conversation persistence under workspace-local metadata, deletion, and exclusion from publishing/sync defaults. Audit existing API-key storage and move desktop secrets to an appropriate credential store before treating cloud setup as production-ready.

The local conversation slice now keeps up to 20 named sessions per workspace and
provider, with new, resume, rename, delete, legacy migration, turn grouping,
message copy/edit/retry, cancellation, and per-turn duration/token metadata.
Search, pin/archive controls, branching, Markdown handoff export, per-message
context labels, stale context warnings, and low-context summary prompting are
implemented. Structured summarize-and-continue and workspace-scoped pinned file
and note context are also implemented.

Provider readiness now has a native diagnostic path for CLI presence/version,
credential state, and protocol availability without exposing secrets. The
remaining gate is the live macOS/Windows provider matrix and release packaging.

Check keyboard navigation, focus restoration, panel resizing, light/dark themes, large-file fallback, provider failures, and offline editing. Measure lazy-loaded editor/chat bundles and typing responsiveness.

## Suggested implementation boundaries

| Area | Responsibility |
| --- | --- |
| Existing code-editor components | Editing, selection, undo, formatting |
| New document/workspace module | Document IDs, tabs, dirty buffers, disk revisions |
| Ask panel and assistant adapter | Conversation state, request lifecycle, endpoint adaptation |
| New context builder | Exact bounded context, provenance, exclusion rules |
| New edit proposal module | Base hashes, diff review, single-transaction apply |
| Native source file commands | Scoped reads/writes and conflict-safe save |
| App integration | Workspace layout, graph navigation, command palette |

Keep new logic out of the already large App.tsx where practical. Final file names are implementation choices.

The source tree now follows these boundaries: `src/features/chat`,
`src/features/editor`, `src/features/graph`, `src/features/notes`,
`src/features/setup`, `src/features/workspace`, and `src/shared`. `App.tsx`,
`main.tsx`, global styles, assets, and generated Vite typings remain at `src/`.

## Validation gates

- Focused tests: context truncation and workspace isolation; abort/late-response races; stale proposal rejection; undo; native path containment and save conflicts.
- Regression: note code fences/frontmatter, external-agent handoff, MCP vault-only writes, branch|HEAD signature, and function-index scheduling.
- Run repo-required lint/check/build and existing MCP/export tests after implementation.
- Desktop smoke on macOS and Windows for file access, save conflicts, keyboard behavior, and request cancellation.
- This planning update changes docs only; implementation tests have not been run.

## Remaining design choices

Proposed defaults are optional in-app chat, existing configurable endpoint, session-only transcript initially, single-file diff application, and explicit source-file saves. Resolve streaming compatibility and native credential storage in Phase 1 discovery. Defer full language services, terminal tools, autonomous editing, and multi-file patch transactions.

Related: [[Code Editor Implementation]], [[Unified Knowledge System]], [[Agent and Human Setup]], [[Agent Skills and Surfaces]], [[Next Steps]], [[Decision Log]].
