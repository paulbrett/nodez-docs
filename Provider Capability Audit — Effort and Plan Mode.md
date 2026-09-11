---
id: nodez-provider-capability-audit-effort-plan-mode
title: Provider Capability Audit — Effort and Plan Mode
type: research
status: draft
created: 2026-09-11
updated: 2026-09-11
tags:
  - ai-chat
  - providers
  - audit
---

# Provider Capability Audit — Effort and Plan Mode

C0 from [[AI Chat Controls and Live Usage Plan]]. Findings from reading all five adapter
sources (`src-tauri/src/{codex,claude,grok,opencode,gemini}.rs`) and, for Codex, the live
installed CLI's own protocol schema (`codex-cli 0.153.4`, via
`codex app-server generate-json-schema`). See
[[2026-09-11-reasoning-effort-and-plan-code-design]] in the app repo for the design this
audit feeds.

Each finding is marked verified present, confirmed absent, or unverified — an installed-but-
unproven mechanism is unverified, never a passing cell.

## Reasoning effort

| Provider | Status | Evidence |
| --- | --- | --- |
| Codex | Verified present | `model/list` RPC returns each `Model` with `supportedReasoningEfforts: [{reasoningEffort, description}]` and `defaultReasoningEffort` — model-discovered, not an invented scale. `turn/start.effort` overrides "this turn and subsequent turns," so switching does not require a reconnect. |
| Claude | Confirmed absent | No effort/thinking field in `claude.rs` or `claude_translate.rs`. Not wired even if the CLI's underlying model supports extended thinking. |
| Grok | Confirmed absent | No effort field in `grok.rs`/`grok_translate.rs`. `session/prompt` carries only session ID and prompt text. |
| OpenCode | Confirmed absent | No effort field in `opencode.rs`. `configOptions` carries only `mode` and `model`. |
| Gemini | Confirmed absent | No effort field in `gemini.rs`, despite Gemini's API having a native `thinkingBudget` concept — unwired. |

## Plan-mode viability (native read-only enforcement)

| Provider | Status | Mechanism |
| --- | --- | --- |
| Codex | Verified present | `sandboxPolicy: {"type": "readOnly"}` — an OS-level sandbox (Seatbelt/Landlock) enforced by the `codex` binary, paired with `approvalPolicy: "never"` so there is no escalation channel. |
| Claude | Verified present | `--permission-mode plan`, a native Claude Code CLI mode the CLI itself enforces by refusing mutating tool calls. |
| Gemini | Verified present | `session/set_mode "plan"`, checked against the CLI's own advertised `modes/availableModes` and **fails closed** if unavailable — the only adapter that refuses to silently degrade to a weaker mode. |
| Grok | Unverified | `--sandbox read-only` is a genuine CLI flag, but the binary is closed-source — nothing in this codebase proves a write is actually denied underneath it. |
| OpenCode | Unverified | Native `"plan"` session mode exists via `session/set_config_option`. (Separately, its `ask` mode is weaker: `ask` and `full-access` share the same native `"build"` mode, with denial enforced client-side by Nodez's own code — not relevant to Plan mode itself, which uses `"plan"`, but worth recording since it's a real gap in the same adapter.) |

## Cross-cutting gap: MCP tool exposure is not mode-aware

All five adapters register the same static `nodez_workspace` MCP tool list — including
`write_note`, `create_note`, `rename_note`, `delete_note` — regardless of permission mode.
Enforcement of "Plan can't write" today depends entirely on the provider's own sandbox/
approval layer; there is no second line of defense at the tool-exposure level. The design doc
closes this by filtering the tool list itself when the effective mode is read-only.

No MCP-exposed way for an agent to launch an orchestration worker exists today (workers are
started only from frontend TypeScript), so there is nothing to gate there.

## What this audit does not cover

Whether Claude, Grok, OpenCode, or Gemini's underlying model/CLI *could* support a reasoning-
effort control if wired — only whether the current Nodez adapter exposes one today (it does
not, for any of the four). Confirming what each actually supports requires the same kind of
live protocol probing done here for Codex, and is scoped as follow-up work per provider.
