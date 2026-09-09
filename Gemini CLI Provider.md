---
id: nodez-gemini-cli-provider
title: Gemini CLI Provider
type: architecture
status: active
created: 2026-09-09
updated: 2026-09-09
tags:
  - ai-chat
  - gemini
  - acp
---

# Gemini CLI Provider

Nodez integrates Gemini CLI as a first-class AI Chat provider through its ACP
stdio mode. The native adapter starts `gemini --acp`, initializes one
window-scoped session, supplies the active-vault Nodez MCP server, translates
standard ACP updates into the shared chat reducer, and terminates the subprocess
on disconnect, provider replacement, or window destruction.

Gemini uses a Google AI Studio API key stored through **Settings → Keys**. Nodez
encodes the secret in its local app-data file and injects it as
`GEMINI_API_KEY` only into the window-scoped Gemini subprocess. The value is
never returned to the frontend or persisted in conversation history. This
headless API-key path replaces the retired Gemini Code Assist individual client
flow that can return “This client is no longer supported.” Install the CLI with
`npm install -g @google/gemini-cli`, save the key, then connect from AI Chat.

The local file is written through a temporary file and receives owner-only
permissions on Unix. Windows replacement currently removes the previous file
before rename; failure recovery is an open task in [[AI Agent Next Steps Handoff]]. Its encoding avoids plain-text storage but does not provide OS-keychain
security. Keys previously stored in a keychain must be entered again.

Models come from `session/new.models.availableModels`; `auto` remains the local
fallback. Selection uses `session/set_model`. Nodez permission modes map to
Gemini `plan`, `default`, and `yolo` through `session/set_mode`. Read-only fails
closed when the installed CLI does not advertise `plan`. Ask mode routes ACP
permission requests to the existing approval cards, and Full access selects
`yolo`. Images use bounded ACP image blocks.

The implementation lives in `src-tauri/src/gemini.rs`, the shared credential
commands in `src-tauri/src/agent_credentials.rs`, and provider records under
`src/features/chat`. The adapter clears conflicting Google auth-selection
environment flags before launch so the stored API key selects Gemini API auth.
Automated frontend, Rust, and build checks pass. Live API-key authentication,
model discovery, prompt streaming, tool approval, and cancellation remain the
next desktop acceptance checks.

Related: [[AI Chat Conversation Experience]], [[Code Editor and AI Chat Plan]],
[[Codex Chat Implementation]].
