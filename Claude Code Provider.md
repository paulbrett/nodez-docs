---
id: nodez-claude-code-provider
title: Claude Code Provider
type: architecture
status: active
created: 2026-09-08
updated: 2026-09-08
tags:
  - claude
  - agents
  - implementation
---

# Claude Code Provider

Claude Code runs as a switchable second agent provider beside the shipped Codex
chat: one AI Chat surface, one live session per window, and a provider picker
that swaps the runtime behind the same transcript, diff, and context UI. This is
the "Claude and Grok adapters are follow-on work" item recorded in
[[Code Editor and AI Chat Plan]].

Spec: `docs/superpowers/specs/2026-09-08-claude-code-provider-design.md`.
Plan: `docs/superpowers/plans/2026-09-08-claude-code-provider.md`.
Protocol captures: `docs/superpowers/plans/2026-09-08-claude-protocol-findings.md`.

## Decisions — 2026-09-08

The user chose each of these during brainstorming.

- **Switchable peer, not a separate panel.** Codex stays; nothing is retired.
- **Raw CLI stdio in Rust.** `claude.rs` drives the installed CLI directly. No
  Node sidecar and no bundled Agent SDK dependency.
- **Claude translates into Codex's protocol.** The existing reducer and panel
  work unchanged. Accepted trade: a vendor protocol is now the app's internal
  event vocabulary, documented in `src-tauri/src/agent_events.rs`.
- **Probe on connect for authentication.** Claude Code has no `account/read`
  equivalent, so Connect runs one cheap request and reads the result. No
  provider API key storage was added.
- **Provider switch behaviour is a Settings choice.** Start fresh by default, or
  carry the conversation as bounded, labelled, text-only context.

## What the spike changed

The protocol capture overturned two assumptions before they were built.

- **No approval channel exists in `--print` mode.** Under `manual`, a write is
  refused outright: the turn still reports success and the file is untouched.
  Claude therefore offers Read-only and Full access only, and the permission
  menu says why. Codex keeps all three modes.
- **Token accounting must count cached context.** A measured turn reported 2
  input tokens against roughly 48,000 tokens of cached context, so the total
  sums the input, output, cache-read, and cache-creation fields.

Two behaviours differ from Codex by nature rather than by choice: Claude fixes
its model and permission mode when the process starts, so changing either
mid-session warns and applies on reconnect, and Stop ends the subprocess because
stream interrupt is unverified.

## Delivery — 2026-09-08

Implemented on `feat/codex-editor-chat` across nine commits.

- `agent_session.rs` holds the spawn, correlation, and teardown plumbing both
  providers share; the Codex refactor onto it changed no test.
- `claude_translate.rs` maps the Claude stream onto the Codex event vocabulary,
  including plan-mode endings and denied permissions.
- `claude_diff.rs` synthesizes unified diffs from `old_string`/`new_string`,
  refusing paths outside the workspace and degrading with a stated reason for
  missing, binary, or oversize files.
- `claude.rs` adds discovery, a 2.1.0 version floor, the connect-time auth
  probe, and vault-scoped MCP restricted to the same thirteen read tools Codex
  exposes.
- The six Tauri commands dispatch on an optional `provider` argument; each
  connect releases the other provider so a switch leaves no orphan subprocess.
- `chatProvider.ts` holds per-workspace provider persistence, the preference-key
  migration, and the carried-context builder.

## Verification

Performed: TypeScript, the production web build, all 28 JavaScript test files,
63 Rust tests, markdown and docs checks. The Codex reducer tests pass with the
file unchanged, which is the evidence the translation is faithful. The recorded
spike session is replayed as a Rust test, so a CLI shape change fails the build
rather than a user's conversation.

Not performed: desktop runs. Neither the fake-CLI harness at
`scripts/fixtures/fake-claude.mjs` nor a real signed-in session has been
exercised through the running app, so connect, streaming, Stop, image
submission, the diff view, and provider switching are unverified end to end.
Windows smoke tests and installer work remain release-stage, as for 0.6.0-rc.1.

## Out of scope

Both providers running at once, cross-provider comparison views, approvals for
Claude, and the Grok adapter. The seam makes Grok cheaper later.

Related: [[Grok Provider]], [[Codex Chat Implementation]], [[Code Editor and AI Chat Plan]],
[[Agent and Human Setup]], [[Decision Log]], [[Next Steps]].
