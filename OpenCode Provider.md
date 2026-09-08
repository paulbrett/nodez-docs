---
id: nodez-opencode-provider
title: OpenCode Provider
type: architecture
status: active
created: 2026-09-08
updated: 2026-09-08
tags:
  - opencode
  - agents
  - acp
---

# OpenCode Provider

Implemented on `feat/codex-editor-chat` on 2026-09-08. OpenCode is the fourth
provider in Nodez AI Chat and uses the Agent Client Protocol exposed by
`opencode acp`.

## Runtime contract

- Nodez launches one window-scoped OpenCode subprocess and correlates ACP
  requests over newline-delimited JSON-RPC.
- Authentication stays in OpenCode's existing data directory. Nodez does not
  store or expose an OpenCode key.
- Nodez gives OpenCode isolated config, state, and cache directories, starts ACP
  with `--pure`, and injects only the active workspace's Nodez MCP during
  `session/new`.
- The ACP model option populates the model selector dynamically. A saved model
  is restored only when the current OpenCode session reports it.
- Read-only maps to OpenCode `plan` mode and denies ACP permission requests.
  Ask mode renders Nodez approval cards. Full access accepts only a one-time
  allow option, so the panel never creates persistent approvals.
- Text, bounded data-URL images, streaming assistant content, tool activity,
  context usage, cancellation, disconnect, and provider switching use the same
  Nodez event contract as the other providers.

## Validation and limits

The ACP handshake, session creation, model configuration, prompt cancellation,
and usage update were exercised with OpenCode 1.18.23. Automated Rust tests cover
permission choices, MCP environment shape, image bounds, provider dispatch, and
ACP translation. The repository TypeScript, production build, script tests, and
Rust suite pass.

OpenCode still needs final interactive desktop acceptance with a funded or free
model that responds promptly. OpenCode currently warns that this Intel Mac lacks
AVX; installing a baseline Bun build is advisable if runtime crashes occur.

Related: [[Codex Chat Implementation]], [[Claude Code Provider]],
[[Grok Provider]], [[Code Editor and AI Chat Plan]], [[Next Steps]].
