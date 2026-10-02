---
id: nodez-ai-provider-updates
title: AI Provider Updates
type: workflow
status: active
created: 2026-10-02
updated: 2026-10-02
tags:
  - ai
  - providers
  - updates
---

# AI Provider Updates

Nodez Settings → AI Chat can check stable releases for Codex, Claude Code, Grok,
OpenCode, Gemini CLI, and ModelArk. Each provider reports its installed and
latest version independently, so one unavailable release service does not hide
the other results.

Updates require an explicit **Update** click and a confirmation showing the
exact command. Nodez blocks an update while that provider has a live session or
another update is running. It runs fixed commands directly, without a shell or
`sudo`, then verifies that the installed version changed.

Nodez updates recognized npm, Homebrew, or native CLI installations. A custom or
ambiguous executable gets **Show instructions** instead. Package-manager
permission problems remain visible for the user to resolve outside Nodez.

## Validation

- Provider version normalization and stable-release parsing have Rust tests.
- Allowlisted updater arguments and the global update lock have Rust tests.
- Frontend state, disabled actions, confirmation copy, and announcements have
  focused Node tests.
- TypeScript checks and the production Vite build pass.

Official references: [Codex CLI](https://developers.openai.com/codex/cli/),
[Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup),
[Grok CLI](https://github.com/xai-org/grok-cli),
[OpenCode](https://opencode.ai/docs/cli/),
[Gemini CLI](https://github.com/google-gemini/gemini-cli), and
[ModelArk CLI](https://docs.byteplus.com/en/docs/ModelArk/arkcli).
