---
id: nodez-ai-account-profiles
title: AI Account Profiles
type: architecture
status: active
created: 2026-09-18
updated: 2026-09-18
tags:
  - agents
  - accounts
  - implementation
---

# AI Account Profiles

Nodez supports named Codex and Claude CLI account profiles. The user authorized
implementation on September 18, 2026 for two accounts with each provider.

## User flow

1. Open Settings → AI Accounts, choose a provider, and add a label such as Personal
   or Work.
2. Click Sign in and complete the provider's browser login using the intended
   account. Closing account management cancels a pending sign-in.
3. Choose the profile from the account picker in chat. Nodez reconnects
   automatically and remembers the selection for that workspace and provider.
4. Finish or stop a running reply, and send or clear pending drafts and
   attachments, before switching.

Default CLI account retains the existing CLI login and legacy chat history.
Named accounts have separate conversation storage, including optional vault
transcript copies. Returning to a profile restores its own conversations.
Continue with context explicitly carries bounded text turns into a new
conversation under the selected account. Approvals, tool results, and native
thread identifiers are not transferred. The connected account email appears in
chat when reported by the provider.

## Storage and identity

- Metadata and provider homes live in the app's local data directory under
  `ai-accounts`, outside the active vault and repository.
- Codex receives a separate `CODEX_HOME` and explicit file credential storage.
  Profile directories are owner-only on Unix. Credentials are managed by Codex;
  they are not stored in webview local storage or encrypted by Nodez.
- Claude receives a separate `CLAUDE_CONFIG_DIR`, which also scopes its macOS
  Keychain entry. Login, status, and runtime use the same canonical path.
- Known inherited API key, OAuth token, and alternate credential selection
  variables are cleared for named profiles. Default CLI sessions retain their
  existing environment.
- A profile is bound to the first email verified by the native adapter. A
  different identity is refused before connecting that profile to a chat. Add
  a separate profile for another identity.
- Native leases prevent changing a profile's login while its chat processes are
  live. Disconnect, process exit, and window closure release those leases.
- Login processes are cancellable and time out after five minutes. Status checks
  do not send model requests. Raw CLI login output is not sent to the UI.

## Implementation

- `src-tauri/src/agent_accounts.rs`: profile metadata, credential environment,
  CLI login/status, identity binding, and account leases.
- `src-tauri/src/codex.rs` and `claude.rs`: profile-aware runtime startup,
  verified email, and transcript storage scope.
- `src/features/chat/AccountManager.tsx`: settings and inline account management.
- `src/features/chat/agentAccounts.ts`: selection and conversation scope helpers.
- `src/features/chat/CodexChatPanel.tsx`: account switching and explicit handoff.
- `src/features/chat/hooks/useAgentSession.ts`: profile ID passed at connect.

New profiles have separate provider configuration and session directories; Nodez
does not copy credentials or configuration from the default CLI home. Account
selection in this slice applies to chat, not orchestration workers. Profile
rename/removal and automatic quota-based account rotation are not implemented.

## Validation

- TypeScript lint and production web build passed.
- Focused JavaScript tests: 72 passed, including account selection, transcript
  separation, provider switching helpers, and stale-event rejection.
- Rust library tests: 122 passed, one existing billed integration test ignored.
- Markdown lint passed. The full docs check is blocked by the existing unsupported
  `status: proposed` in Workspace Improvements Plan.
- Desktop browser login with real accounts, macOS Keychain behavior with two real
  Claude accounts, and Windows login remain manual smoke checks. Automated tests
  do not establish those flows as release-certified.

Related: [[Claude Code Provider]], [[Codex Chat Implementation]],
[[AI Workspace Delivery]], [[Agent and Human Setup]].
