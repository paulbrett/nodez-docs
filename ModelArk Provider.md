---
id: nodez-modelark-provider
title: ModelArk Provider
type: architecture
status: active
created: 2026-09-14
updated: 2026-09-15
tags:
  - ai-chat
  - modelark
  - byteplus
---

# ModelArk Provider

Nodez integrates BytePlus ModelArk as its sixth AI Chat provider through the
installed `arkcli` executable. Unlike Codex App Server and the ACP providers,
Ark CLI exposes structured streaming from `arkcli +chat`. The native adapter
translates its NDJSON response events into Nodez's shared turn and message event
contract, scopes each process to a window lane, and terminates it on Stop,
disconnect, provider replacement, or window destruction.

The default profile is `coding-plan_ap-southeast-1_personal`; deployments may
override it with `NODEZ_MODELARK_PROFILE`. Nodez does not copy or store the Ark
API key. Ark CLI resolves its own profile and masked credential state. The
working profile reports an active key, an `auto` default, and 11 invocable text
models. The model picker includes Auto, Dola Seed 2.0 Pro/Lite/Code, ByteDance
Seed Code, GLM 5.2/5.1, DeepSeek V4 Flash/Pro, Kimi K2.5, and GPT OSS 120B.

ModelArk supports Read-only and Full access in Nodez, the same two modes as
Claude. Read-only omits workspace functions; Full access exposes the
adapter's bounded file, search, and command tools. Ark CLI does not provide a
native per-tool approval protocol, so ModelArk does not emit approval
requests — an "Ask" mode was offered through 2026-09-18, but it silently
behaved exactly like Full access (tools ran with no prompt), so it was
removed rather than left to promise a confirmation step that could never
happen. Permissions and conversation continuity are independent: multi-turn
chat uses `--store` and passes each completed response ID back as
`--previous-response-id` on the next turn.

For tool-enabled turns, the adapter decodes Ark's JSON-string function
arguments, executes each completed call once, and continues the same turn for
up to 64 response/tool rounds. Because the installed Ark CLI does not accept
structured `function_call_output` input, tool results are sent as a clearly
labelled continuation prompt while retaining the stored response ancestry.
Further tool calls are therefore supported, and the adapter rejects malformed
arguments, missing call IDs, and tools in Read-only mode. Stop/interrupt marks
the turn interrupted, kills the active process, and prevents follow-up rounds.

It supports streamed text and Stop. Images are not exposed in this adapter even
though Ark CLI supports file inputs independently.

Live acceptance on 2026-09-15 used Ark CLI 1.0.27 and the explicit Coding Plan
profile. A live read-only multi-round probe read `package.json` and
`AGENTS.md` in separate tool responses and reached the exact final marker
`NODEZ-CONTINUED`. Rust unit tests cover argument decoding, call
deduplication, compatibility result prompts, continuation IDs, and the
multi-round smoke test. The actual CLI emits underscore-form event names such as
`response_output_text_delta` and `response_completed`; the adapter accepts
those names and the documented dotted variants.

Implementation lives in `src-tauri/src/modelark.rs`, with dispatch and teardown
in `src-tauri/src/codex.rs` and `src-tauri/src/lib.rs`. Provider capabilities,
models, onboarding, readiness, chat, and orchestration selection use the shared
frontend provider records.

Related: [[AI Chat Conversation Experience]], [[Code Editor and AI Chat Plan]],
[[AI Agent Next Steps Handoff]], [[September 9 Improvements]].
