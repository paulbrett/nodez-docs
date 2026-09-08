---
id: nodez-ai-chat-conversation-experience-plan
title: AI Chat Conversation Experience Implementation Plan
type: roadmap
status: active
created: 2026-09-08
updated: 2026-09-08
tags:
  - ai-chat
  - implementation
---

# AI Chat Conversation Experience Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use
> `superpowers:subagent-driven-development` (recommended) or
> `superpowers:executing-plans` to implement this plan task-by-task. Steps use
> checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the latest provider conversation resumable and recoverable, expose
its context and usage clearly, reduce approval friction, and split the chat panel
into focused testable modules.

**Architecture:** Keep `CodexChatPanel` as the composition shell while extracting
provider policy, popover coordination, session lifecycle, persistence, messages,
composer, context, usage, and approvals. Persist one normalized conversation per
workspace/provider and retain opaque native session IDs only where supported;
otherwise resume through bounded labelled replay.

**Tech Stack:** React 19, TypeScript, Tauri 2, Rust, Vitest-compatible Node test
fixtures, existing Codex-shaped event reducer.

**Spec:** `Documents/Projects/Nodez/AI Chat Conversation Experience.md`

## Global constraints

- Preserve workspace, window, provider, thread, and turn stale-event rejection.
- Never persist image data, secrets, pending requests, or approval rules.
- Keep source containment, file-size bounds, MCP restrictions, and permission-mode
  semantics unchanged.
- Keep one recent conversation per workspace/provider; do not add a conversation
  library or branching.
- Edit only the latest user message.
- Never estimate provider cost from incomplete usage or unversioned prices.
- Session command rules expire on disconnect, provider/workspace change, window
  close, or app restart.
- Release packaging, publication, commit, and push are separate explicit actions.

---

### Task 1: Centralize provider capabilities

**Files:**

- Create: `src/chatProviderCapabilities.ts`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/chat-provider.test.mjs`

**Interfaces:**

- Consumes: `ChatProvider` from `src/chatProvider.ts`.
- Produces: `PermissionMode`, `ProviderCapabilities`,
  `PROVIDER_CAPABILITIES`, and `providerCapabilities(provider)`.

- [ ] **Step 1: Add failing capability assertions**

```ts
const { providerCapabilities } = await import('../src/chatProviderCapabilities.ts');
assert.deepEqual(providerCapabilities('claude').permissionModes, ['read-only', 'full-access']);
assert.equal(providerCapabilities('grok').supportsImages, false);
assert.equal(providerCapabilities('codex').supportsApprovals, true);
assert.equal(providerCapabilities('opencode').reconnectOnModelChange, true);
```

- [ ] **Step 2: Run the focused test and confirm the missing-module failure**

Run: `node --experimental-strip-types --test scripts/chat-provider.test.mjs`

- [ ] **Step 3: Implement the typed immutable capability map**

```ts
export type PermissionMode = 'read-only' | 'ask' | 'full-access';
export type ProviderCapabilities = {
  permissionModes: readonly PermissionMode[];
  supportsImages: boolean;
  supportsApprovals: boolean;
  supportsNativeResume: boolean;
  reconnectOnModelChange: boolean;
  reconnectOnPermissionChange: boolean;
};
export function providerCapabilities(provider: ChatProvider): ProviderCapabilities {
  return PROVIDER_CAPABILITIES[provider];
}
```

- [ ] **Step 4: Replace provider ternaries in connection instructions, image
  controls, permissions, and reconnect behavior with capability lookups**

- [ ] **Step 5: Run provider tests and TypeScript**

Run: `node --experimental-strip-types --test scripts/chat-provider.test.mjs && npm run lint`

### Task 2: Replace hand-wired menu cross-closing

**Files:**

- Create: `src/hooks/useDismissablePopover.ts`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/dismissable-popover.test.mjs`

**Interfaces:**

- Produces: `useDismissablePopoverGroup(ids)` returning `register(id)`,
  `onToggle(id, open)`, and `closeAll()`.
- The hook closes other registered `<details>` elements, closes on outside pointer
  or focus, closes on Escape, and returns focus to the originating summary.

- [ ] **Step 1: Extract pure `nextOpenPopover(current, toggled, open)` and test
  exclusive-open behavior**

```ts
assert.equal(nextOpenPopover('context', 'model', true), 'model');
assert.equal(nextOpenPopover('model', 'model', false), null);
```

- [ ] **Step 2: Run the test and confirm it fails before implementation**

Run: `node --experimental-strip-types --test scripts/dismissable-popover.test.mjs`

- [ ] **Step 3: Implement the hook with one document listener pair and Escape
  handling**

- [ ] **Step 4: Replace the four refs and repeated `onToggle` cross-closing blocks
  in `CodexChatPanel`**

- [ ] **Step 5: Run focused test, lint, and build**

Run: `node --experimental-strip-types --test scripts/dismissable-popover.test.mjs && npm run lint && npm run build`

### Task 3: Add normalized recent-conversation storage

**Files:**

- Create: `src/chatConversationStore.ts`
- Modify: `src/CodexChatPanel.tsx`
- Modify: `src/codexChat.ts`
- Test: `scripts/chat-conversation.test.mjs`

**Interfaces:**

- Produces: `PersistedConversationV1`, `conversationKey(scope, provider)`,
  `loadConversation(storage, scope, provider)`, and
  `saveConversation(storage, scope, provider, conversation)`.
- `scope` is the existing stable workspace identity passed from `App.tsx`.
- Persisted items include text turns only, maximum 100 items and 200,000 text
  characters total; `imageUrls` are removed.

- [ ] **Step 1: Write storage tests for provider isolation, workspace isolation,
  invalid JSON, version rejection, item bounds, and image removal**

```ts
saveConversation(store, 'vault-a:repo-a', 'codex', conversation);
assert.equal(loadConversation(store, 'vault-a:repo-a', 'claude'), null);
assert.equal(loadConversation(store, 'vault-b:repo-a', 'codex'), null);
assert.equal(loadConversation(store, 'vault-a:repo-a', 'codex')?.items[0].imageUrls, undefined);
```

- [ ] **Step 2: Run tests and confirm the missing-module failure**

Run: `node --experimental-strip-types --test scripts/chat-conversation.test.mjs`

- [ ] **Step 3: Implement parsing, normalization, and bounded serialization without
  throwing on unavailable storage**

- [ ] **Step 4: Add `restoreConversation(state, record)` to initialize visible
  text turns, usage, and thread metadata with `busy=false`, no approvals, and no
  active turn**

- [ ] **Step 5: Pass a stable `conversationScope` from `App.tsx`; load on
  scope/provider change and save after stable completed turns**

- [ ] **Step 6: Keep the existing `.nodez/chat-history` display control working
  during migration; do not send it automatically**

- [ ] **Step 7: Run store/reducer tests, lint, and build**

Run: `node --experimental-strip-types --test scripts/chat-conversation.test.mjs scripts/codex-chat.test.mjs && npm run lint && npm run build`

### Task 4: Add latest-message edit and retry state transitions

**Files:**

- Modify: `src/codexChat.ts`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/codex-chat.test.mjs`

**Interfaces:**

- Produces: `latestEditableUserMessage(items)`, `replaceLatestTurn(items, text)`,
  and `retryablePrompt(items)`.
- A user message and its following activity/assistant items form the latest turn.

- [ ] **Step 1: Add reducer-helper tests for a successful turn, failed turn,
  command activity between messages, no user prompt, and bounded replacement**

```ts
assert.equal(latestEditableUserMessage(items)?.text, 'original');
assert.deepEqual(replaceLatestTurn(items, 'corrected').map(item => item.text), ['corrected']);
assert.equal(retryablePrompt(items)?.text, 'original');
```

- [ ] **Step 2: Run the focused tests and confirm missing exports fail**

Run: `node --experimental-strip-types --test scripts/codex-chat.test.mjs`

- [ ] **Step 3: Implement pure helpers and preserve earlier completed turns**

- [ ] **Step 4: Add Edit to the latest user message and Retry to the latest
  assistant/error turn; disable both while busy or requests are pending**

- [ ] **Step 5: Edit loads text into the composer; submit replaces the latest turn
  and starts a fresh provider turn. Retry reuses the recorded latest prompt**

- [ ] **Step 6: Add hover/focus action styling and accessible labels**

- [ ] **Step 7: Run focused tests, lint, and build**

Run: `node --experimental-strip-types --test scripts/codex-chat.test.mjs && npm run lint && npm run build`

### Task 5: Extract the agent-session lifecycle

**Files:**

- Create: `src/hooks/useAgentSession.ts`
- Modify: `src/CodexChatPanel.tsx`
- Modify: `src/codexChat.ts`
- Test: `scripts/agent-session.test.mjs`

**Interfaces:**

- Produces: `UseAgentSessionOptions` and `AgentSessionController` with `connect`,
  `disconnect`, `request`, `sendTurn`, `stop`, `respond`, `state`, `connection`,
  and `connecting`.
- Consumes provider capabilities and restored conversation metadata.

- [ ] **Step 1: Extract and test pure `acceptAgentEvent(identity, envelope)` for
  workspace, provider, session, thread, turn, and completed-turn mismatches**

- [ ] **Step 2: Run the focused test and confirm missing exports fail**

Run: `node --experimental-strip-types --test scripts/agent-session.test.mjs`

- [ ] **Step 3: Move listener, correlation refs, request, connect, disconnect,
  login completion, Stop, and approval response into the hook without changing
  native commands**

- [ ] **Step 4: Restore the visible recent conversation before connect. Attempt
  native resume only for a capability that supports it; otherwise attach bounded
  replay once to the next turn**

- [ ] **Step 5: On failed native resume, retain visible history, clear the opaque
  ID, queue bounded replay, and expose a recoverable warning**

- [ ] **Step 6: Run session/reducer/provider tests, lint, build, and Rust tests**

Run: `node --experimental-strip-types --test scripts/agent-session.test.mjs scripts/chat-conversation.test.mjs scripts/codex-chat.test.mjs scripts/chat-provider.test.mjs && npm run lint && npm run build && cargo test --manifest-path src-tauri/Cargo.toml --lib`

### Task 6: Extract composer and add visible context chips

**Files:**

- Create: `src/components/chat/ChatComposer.tsx`
- Create: `src/components/chat/ChatContextChips.tsx`
- Create: `src/chatContextSelection.ts`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/chat-context-selection.test.mjs`

**Interfaces:**

- Produces `ContextSelection`, `ContextChip`, `addContext`, `removeContext`, and
  `buildContextManifest` with stable IDs and content hashes.

- [ ] **Step 1: Test chip deduplication, removal, stale flags, selection/file
  exclusivity, and manifest hashing**
- [ ] **Step 2: Implement pure context-selection helpers**
- [ ] **Step 3: Extract the composer without changing send keys, images, model,
  permissions, or Stop**
- [ ] **Step 4: Render selected context as removable chips above the textarea**
- [ ] **Step 5: Run context/chat tests, lint, and build**

### Task 7: Add the `@` file and note picker

**Files:**

- Create: `src/components/chat/ChatMentionPicker.tsx`
- Create: `src/chatMentions.ts`
- Modify: `src/components/chat/ChatComposer.tsx`
- Test: `scripts/chat-mentions.test.mjs`

**Interfaces:**

- Produces `mentionQuery(value, cursor)`, `filterMentionCandidates`, and a picker
  returning a `ContextChip`; it does not insert a provider-visible path token.

- [ ] **Step 1: Test trigger boundaries, ranking, duplicate candidates, Escape,
  and selection replacement**
- [ ] **Step 2: Implement pure mention parsing and ranking**
- [ ] **Step 3: Implement keyboard navigation with active-descendant semantics**
- [ ] **Step 4: Resolve the selection through existing bounded note/file context
  builders and add the resulting chip**
- [ ] **Step 5: Run mention/context tests, lint, and build**

### Task 8: Replace raw graph JSON with readable context

**Files:**

- Create: `src/components/chat/ChatContextPreview.tsx`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/codex-context.test.mjs`

**Interfaces:**

- Consumes existing `GraphContextEntry[]`.
- Produces readable rows while leaving `buildGraphContext` payload bounds and
  provenance unchanged.

- [ ] **Step 1: Extend tests for label, relation, provenance, confidence,
  source path/line, and one stale warning**
- [ ] **Step 2: Implement the preview component and remove the JSON `<pre>`**
- [ ] **Step 3: Verify citations still open the exact note/file/line**
- [ ] **Step 4: Run context tests, lint, and build**

### Task 9: Extract messages and add copy/search

**Files:**

- Create: `src/components/chat/ChatMessageList.tsx`
- Create: `src/components/chat/ChatMessageItem.tsx`
- Create: `src/chatTranscriptSearch.ts`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/chat-transcript-search.test.mjs`

**Interfaces:**

- Produces `searchTranscript(items, query)` returning item IDs and match ranges.

- [ ] **Step 1: Test case-insensitive search, empty query, command output,
  clipped results, and stable ordering**
- [ ] **Step 2: Implement search and extract existing item rendering unchanged**
- [ ] **Step 3: Add transcript search with next/previous controls and match count**
- [ ] **Step 4: Add per-message Copy with clipboard error feedback**
- [ ] **Step 5: Run transcript/Markdown/chat tests, lint, and build**

### Task 10: Normalize usage and reliable cost display

**Files:**

- Create: `src/chatUsage.ts`
- Create: `src/components/chat/ChatUsage.tsx`
- Modify: `src/codexChat.ts`
- Modify: provider translators under `src-tauri/src/`
- Test: `scripts/chat-usage.test.mjs`

**Interfaces:**

- Produces `NormalizedUsage`, `normalizeUsage(provider, event)`, and
  `displayableCost(usage, modelPricing)`.

- [ ] **Step 1: Test input/output/cache totals, remaining context, missing limit,
  provider-reported cost, exact versioned price, and unknown price**
- [ ] **Step 2: Implement normalized usage without inferred currency values**
- [ ] **Step 3: Map each provider's available usage fields in its translator**
- [ ] **Step 4: Render compact usage in connection/history and low-context state
  near the composer**
- [ ] **Step 5: Run usage/provider fixtures, lint, build, and Rust tests**

### Task 11: Add session-scoped similar-command approvals

**Files:**

- Create: `src/chatSessionApprovals.ts`
- Create: `src/components/chat/ChatApprovalCard.tsx`
- Modify: `src/hooks/useAgentSession.ts`
- Modify: `src/CodexChatPanel.tsx`
- Test: `scripts/chat-session-approvals.test.mjs`

**Interfaces:**

- Produces `SessionApprovalRule`, `ruleFromApproval(request)`, and
  `matchesSessionRule(request, rule)` using provider-proposed prefixes only.

- [ ] **Step 1: Test exact prefix boundaries, shell-token mismatches, missing
  prefix, non-command requests, read-only mode, and rule expiry**
- [ ] **Step 2: Implement fail-closed rule normalization and matching**
- [ ] **Step 3: Extract approval rendering and add Allow similar commands this
  session only when a valid command rule exists**
- [ ] **Step 4: Auto-answer matching later requests through the existing response
  channel and append visible automatic-approval activity**
- [ ] **Step 5: Clear rules in every disconnect/provider/workspace/unmount path**
- [ ] **Step 6: Run approval/provider tests, lint, build, and Rust tests**

### Task 12: Complete regression and desktop acceptance

**Files:**

- Modify: `scripts/codex-protocol-fixture.mjs`
- Modify: `scripts/fixtures/fake-claude.mjs`
- Modify: `scripts/codex-protocol-fixture.mjs`
- Modify: `Documents/Projects/Nodez/AI Workspace Delivery.md`
- Modify: `Documents/Projects/Nodez/AI Chat Conversation Experience.md`

**Interfaces:** None; this task verifies the complete user-visible contract.

- [ ] **Step 1: Run all TypeScript, JavaScript, Markdown, frontend-build, and Rust
  checks from `AGENTS.md`**
- [ ] **Step 2: Run controlled fixtures for connect, streaming, Stop, retry,
  restoration, stale events, questions, diffs, one-time approval, and session
  approval on every supported provider**
- [ ] **Step 3: Run a live Codex handshake and macOS desktop smoke covering close,
  reopen, edit, retry, chips, search, usage, and approval expiry**
- [ ] **Step 4: Record Windows-only acceptance as pending unless a Windows host is
  available; do not claim it passed without evidence**
- [ ] **Step 5: Update the two vault delivery notes with exact evidence and rebuild
  the Nodez graph when durable docs freshness is required**
