---
id: nodez-codex-chat-implementation
title: Codex Chat Implementation
type: architecture
status: active
created: 2026-09-07
updated: 2026-09-08
tags:
  - codex
  - implementation
---

# Codex Chat Implementation Plan

Goal: an optional Codex chat panel with project context, streaming, command activity, diffs, approval UI, and cancellation.

Architecture: React panel → Tauri commands → per-window Codex App Server subprocess over newline JSON. Codex owns authentication and conversation storage; Nodez never copies its credentials. Start read-only with on-request approvals routed to the human. Native bridge controls allowed RPC methods and session identity. Nodez MCP is scoped to the selected vault with read tools only for this first integration.

Tech stack: existing React/TypeScript/Tauri/Rust, installed Codex CLI. No hosted service.

Spec: [[Code Editor and AI Chat Plan]]. User authorized Codex-first implementation on 2026-09-07.

## Constraints

- Code stays in app repo; planning stays in this vault.
- Source indexing and MCP retain read-only source boundaries.
- No change to branch|HEAD re-indexing or function scan lifecycle.
- No new provider API key storage.
- Closing the panel/window or switching workspace ends the native session.
- Unknown server approval/tool requests fail explicitly rather than hang.
- Frontend receives only its own window/session events.
- Existing thin Ask remains available.

## Task 1 — Native transport and session lifecycle

Files: src-tauri/src/codex.rs; command registration in src-tauri/src/lib.rs.

- [ ] Spawn installed codex app-server with piped stdio; discover executable through override, PATH, and common app/CLI locations.
- [ ] Route responses by request ID with timeout; emit notifications/approval requests to the owning window with session ID.
- [ ] Initialize client, read account state, list models, and start conversation on explicit Connect.
- [ ] Canonicalize workspace/vault; override sandbox and human approval policy; disable inherited MCP servers and enable only active-vault read tools.
- [ ] Validate allowed RPC methods, thread IDs, and approval responses natively.
- [ ] Kill subprocess tree and fail pending requests on disconnect/window destruction.
- [ ] Rust tests for response routing, transport failure, approval validation, and root validation.

## Task 2 — Chat state and panel

Files: src/codexChat.ts, src/CodexChatPanel.tsx, src/codexChat.css.

- [ ] Normalize streaming items, output, turn completion/errors, diff updates, and approval requests.
- [ ] Bound visible output and ignore mismatched thread/session events.
- [ ] Connect/account guidance, model choice, current-note context toggle, conversation transcript, composer, Stop, disconnect.
- [ ] Render command output, file changes, diffs, human approvals, and user-input requests.
- [ ] Tests for streamed/final item reconciliation, errors, foreign-thread events, and context bounds.

## Task 3 — Integration and verification

Files: src/App.tsx, src/commands.ts, AGENTS.md.

- [ ] Add optional Codex entry point beside existing Ask and in command palette.
- [ ] Remount panel on vault/source switch; include current unsaved note context explicitly.
- [ ] Run lint/check/build, existing MCP/export tests, and Rust tests.
- [ ] Exercise installed Codex handshake and UI with controlled protocol fixture; document whether live model run and cross-platform desktop smoke were performed.
- [ ] Update vault results and rebuild graph.

## Permission modes — 2026-09-07

The user explicitly requested VS Code-style permission choices including Full access.
This supersedes the fixed read-only/on-request runtime policy above.

- Read-only (default): `read-only` sandbox with `never` approval policy; no escalation.
- Ask for approval: `read-only` sandbox with `on-request` approval policy.
- Full access: `danger-full-access` sandbox with `never` approval policy. Commands,
  network access, and filesystem writes can run without approval prompts, including
  outside the workspace. The panel explains this before connection.
- The selected mode applies to the next message/session request and is stored as
  a local UI preference. The custom permission menu disables changes during an
  active response.
- Active-vault MCP tool restrictions and source-preview containment remain enforced.
  Instructions still require explicit requests for commit/push/publish/delete;
  these are behavioral instructions, not an OS restriction in Full access.

Validation: installed Codex JSON schema confirms the runtime enum values;
TypeScript and production build passed. Desktop permission behavior still needs
live verification. The broader repository workspace and reviewed-edit milestones
remain pending.

## Delivery update

See [[AI Workspace Delivery]] for the 0.6.0-rc.1 implementation checklist, live
macOS and protocol-fixture evidence, automated test results and remaining release
gates. The task lists above are the original plan; the delivery note distinguishes
implemented behavior from unperformed cross-platform acceptance tests.

## Workspace UI update — 2026-09-08

The Codex integration now uses one persistent chat surface across Notes, Code,
and the expanded AI Chat tab. Notes and Code can show it as a resizable right
sidebar; opening AI Chat expands the same mounted conversation instead of
starting another session. Chat width is clamped, stored per workspace, and has
a compact-width fallback.

The native App Server bridge supports account connection, model discovery,
streaming activity, Stop, disconnect, approval and user-question cards, command
activity, diffs, image attachments, and workspace-scoped event filtering. The
disconnected view hides the composer and transcript and presents a centered
Connect Codex flow with setup instructions.

The composer now has:

- a custom context/image menu;
- custom permission and model menus without native select arrows;
- Read-only, Ask for approval, and Full access modes;
- Enter-to-send by default, Shift+Enter for a new line, and a persisted Settings
  toggle for Command/Ctrl+Enter behavior;
- syntax-highlighted Markdown code blocks and image submission.

The old right document inspector was removed. Outline, Backlinks, Outgoing, and
Tags now live as full-width collapsible explorer sections. Markdown repository
files get an Outline section; non-Markdown code files do not. Notes and
repository trees use matching icons, typography, compact rows, and edge-to-edge
sidebar layout.

The repository workspace includes source tabs, recent files, language-aware
CodeMirror highlighting, separate code theme/font/size settings, dirty state,
explicit save/reload actions, external-change handling, bounded contained file
access, atomic conflict-aware saves, and reviewed single-file AI edit proposals.
Repository editing is enabled by default and can be disabled in Settings.

Validation completed on macOS during development: TypeScript lint, production
web build, focused chat/editor/layout tests, the broader JavaScript suite, and
Rust library tests. Release packaging, Windows desktop smoke, installer checks,
and final live acceptance remain release-stage work and are intentionally
deferred.

## Multi-provider — 2026-09-08

The chat surface is no longer Codex-only. Claude Code ships as a switchable
provider behind the same panel ([[Claude Code Provider]]), and Grok is designed
over ACP but not implemented ([[Grok Provider]]). Codex remains the only
provider whose approval cards were reachable until Grok lands.
