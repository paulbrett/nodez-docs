---
id: nodez-ai-chat-conversation-experience
title: AI Chat Conversation Experience
type: architecture
status: active
created: 2026-09-08
updated: 2026-09-08
tags:
  - ai-chat
  - agents
  - context
  - persistence
---

# AI Chat Conversation Experience

Nodez should make its shared Codex, Claude, Grok, and OpenCode chat feel like a
durable developer tool. The next iteration adds resumable recent conversations,
message recovery, visible context, transcript navigation, usage information, and
lower-friction approvals while splitting the current monolithic panel along clear
runtime and UI boundaries.

Related: [[AI Workspace Delivery]], [[Code Editor and AI Chat Plan]],
[[Codex Chat Implementation]], [[Claude Code Provider]], [[Grok Provider]].

## Accepted decisions

- Keep one most-recent conversation per workspace and provider. Do not add a
  conversation library, naming, or branching in this phase.
- Editing is available only for the latest user message. If that message already
  has a response, edit removes the response and resends the turn.
- Retry reruns the latest assistant turn, including failed turns, without making
  the user retype the prompt.
- Approval cards offer **Allow once**, **Allow similar commands this session**,
  and **Deny** when the provider protocol supports approval requests.
- Similar-command approval uses the provider-proposed command prefix. It is kept
  in memory and expires on disconnect, provider change, workspace change, window
  close, or app restart. It is never persisted as a global approval.
- Deliver through incremental extraction. Avoid a parallel rewrite and avoid
  adding more feature logic directly to `CodexChatPanel.tsx`.

## Scope

The complete experience includes:

1. Provider capability records and one reusable popover controller.
2. A focused agent-session hook for connection, RPC, streaming, cancellation,
   stale-event rejection, and persistence coordination.
3. Resumption of the most recent provider-bound conversation.
4. Latest-message edit, resend, retry, and per-message copy.
5. Composer context chips and an `@` file picker.
6. A readable graph-context preview with relation, provenance, confidence, and
   source location instead of raw JSON.
7. Transcript search.
8. Token and context-window usage. Cost appears only when reliable provider usage
   and price data are available; unknown cost is omitted rather than estimated.
9. Session-scoped approval of similar commands.

Multiple saved conversations, conversation branching, persistent approval rules,
cross-provider transcript continuation, and provider-cost estimates are outside
this scope.

## Architecture

`CodexChatPanel` becomes a composition shell. It owns workspace-level placement
and passes stable inputs to focused modules:

- `chatProviderCapabilities.ts` describes display name, connect instructions,
  supported permission modes, images, native resume, usage fields, approvals,
  and model-change behavior for each provider.
- `useAgentSession.ts` owns native connection lifecycle, event subscription,
  request correlation, Stop, disconnect, stale-event guards, provider-native
  session identifiers, and reducer dispatch.
- `chatConversationStore.ts` owns the versioned workspace/provider storage key,
  normalization, bounded persistence, migration from display-only history, and
  restoration of the latest conversation.
- `ChatMessageList.tsx` and `ChatMessageItem.tsx` render turns, activity, diffs,
  questions, copy, edit, and retry controls.
- `ChatComposer.tsx` owns draft text, images, context chips, the `@` picker,
  permission/model controls, and send-key behavior.
- `ChatContextPreview.tsx` renders note, file, selection, and graph context in a
  readable form and generates the bounded payload sent to providers.
- `ChatUsage.tsx` renders reported token/context data and optional reliable cost.
- `ChatApprovalCard.tsx` renders provider-supported approval choices and session
  rule feedback.
- `useDismissablePopover.ts` replaces the four menu refs and repeated cross-close
  handlers with one accessible outside-click/Escape/focus-return contract.

Shared protocol and reducer types remain in `codexChat.ts` initially. Provider
native bridges remain separate in Rust. This refactor does not change source-root
write containment, MCP exposure, or the existing permission-mode meanings.

## Conversation identity and persistence

The storage scope is `window + workspace + provider`. A persisted record contains
a schema version, workspace identity, provider, normalized messages, timestamps,
the last selected model, usage snapshot, and an optional opaque native session ID.
Image data, approval rules, pending requests, secrets, and raw provider events are
never stored.

On connect, Nodez loads the latest compatible record. If the provider supports
native resume, the bridge attempts that opaque ID. If resume is rejected or not
supported, Nodez starts a fresh native session and sends a bounded, clearly
labelled transcript before the next user message. Restoration failure leaves the
saved transcript visible, reports that live context could not be restored, and
allows a fresh turn.

Persistence writes only after stable reducer transitions and uses the existing
serialized write queue. A conversation is never restored across a different
workspace or provider. Disconnect preserves the completed conversation but clears
all live correlation IDs and temporary approvals.

## Message actions

Every completed user or assistant message exposes Copy on hover and keyboard
focus. The latest user message also exposes Edit. Editing loads its text into the
composer; confirming removes the associated assistant response and subsequent
activity for that turn, then sends a new turn. Images from an old message are not
silently restored because image payloads are intentionally not persisted.

The latest assistant message exposes Retry. Retry uses the immediately preceding
user payload and its recorded context manifest. If referenced context is stale or
missing, Nodez shows the changed entries before sending and requires the user to
send from the composer. Retry is disabled during streaming or while approval and
question requests are pending.

## Context experience

Selected context remains visible as removable chips in the composer: current
note, current file, selection, graph neighbors, and explicitly mentioned files.
Each chip opens its preview and reports stale or missing content. The `+` menu
continues to add images and broader context sources.

Typing `@` opens a keyboard-navigable picker scoped to the attached repository and
notes vault. Choosing an entry inserts a context chip, not a fragile path token in
the prompt. Duplicate selections collapse to one chip. Path containment, file-size
limits, and context bounds remain enforced by the existing native and context
builders.

Graph preview rows show the related node, relationship, provenance, confidence,
and source path/line. Stale graph state appears once above the rows. The provider
payload remains structured and bounded even though the UI no longer displays raw
JSON.

## Usage and cost

The capability record maps provider usage events into normalized input, output,
cache-read, cache-write, total, context-limit, and optional cost fields. The panel
shows compact context usage in the connection/history menu and a warning near the
composer when remaining context is low.

Cost is displayed only when the provider reports a monetary amount or Nodez has an
exact versioned price for the selected model and all billed token categories.
Otherwise Nodez shows tokens without a currency value. OpenCode free models may
show `Free` from the approved model catalog.

## Session approvals

When an approval request includes a safe provider-proposed command prefix, the
card can allow similar commands for the current session. The session hook stores
the normalized prefix in memory and auto-approves later matching requests through
the same native response channel. The UI records each automatic approval as
command activity.

Requests without a usable prefix offer Allow once and Deny only. File-change and
non-command approvals remain one-time unless a future provider protocol supplies
a narrowly scoped rule. Read-only mode never creates session rules. Full access
continues to bypass approval prompts according to its existing definition.

## Error handling

- Failed sends retain the user message and expose Retry.
- Native-resume failure falls back to bounded replay and reports the fallback.
- Corrupt or incompatible persisted records are ignored without blocking connect.
- Provider changes close the live session, clear temporary approval rules, and
  restore only that provider's workspace-scoped recent conversation.
- Late events remain rejected by window, session, thread, turn, provider, and
  workspace identity.
- Context changed since the original turn blocks automatic retry and shows which
  context entries changed.
- Clipboard failures leave content intact and show a local error/toast.

## Implementation checkpoint — 2026-09-08

Slices 1–3 are implemented on `feat/codex-editor-chat`: provider capabilities,
dismissable popovers, the native session hook, workspace/provider-scoped recent
conversation restoration with bounded replay, latest-turn edit and retry, context
chips, the `@` file/note picker, readable graph provenance, composer extraction,
transcript search, per-message copy, and reported context-window usage. The code
editor also keeps focus across parent context updates and applies its configured
font family and size directly to CodeMirror.

Slice 4 remains: narrowly scoped in-memory command-prefix approvals and final
live multi-provider regression checks. Native provider responses still accept
one-time approval decisions only, and no approval rule is persisted.

## Delivery slices

### Slice 1 — foundation and recovery

Extract provider capabilities and dismissable popovers, introduce the session and
conversation-store seams, resume the most recent provider conversation, and add
latest-message edit/resend/retry. This slice must preserve current connect,
streaming, Stop, approvals, questions, diffs, images, and model selection.

### Slice 2 — visible context

Extract the composer, add removable chips and the `@` picker, and replace raw graph
JSON with readable rows. Preserve exact context bounds and provenance.

### Slice 3 — transcript and usage

Extract message rendering, add per-message copy and transcript search, normalize
usage, and show tokens/context plus reliable cost when available.

### Slice 4 — session approvals and cleanup

Add similar-command rules, record automatic approvals, remove superseded panel
paths, and complete live multi-provider regression testing.

## Verification

Each slice adds focused reducer/store/component tests before implementation. The
full acceptance matrix covers:

- restore after panel close and app restart;
- provider and workspace isolation;
- native-resume success, rejection, and bounded-replay fallback;
- edit latest message, retry success/failure, and cancellation;
- stale events after disconnect, provider switch, and workspace switch;
- context chips, `@` keyboard navigation, deduplication, bounds, and stale files;
- readable graph provenance and stale-state rendering;
- transcript search and clipboard failure;
- usage with cached tokens, missing limits, exact cost, and unknown cost;
- allow once, session-prefix allow, mismatch denial, and rule expiry;
- existing streaming, approvals, questions, image, Markdown, diff, MCP-boundary,
  editor, graph, and Git-signature regression suites.

Before release, run TypeScript, production build, focused JavaScript tests, Rust
library tests, a controlled protocol fixture for every provider, a live Codex
handshake, and macOS/Windows desktop smoke tests. Release packaging remains a
separate explicitly requested action.
