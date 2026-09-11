---
id: nodez-ai-chat-controls-live-usage-plan
title: AI Chat Controls and Live Usage Plan
type: roadmap
status: draft
created: 2026-09-11
updated: 2026-09-11
tags:
  - ai-chat
  - tokens
  - planning
  - usability
---

# AI Chat Controls and Live Usage Plan

## Outcome and scope

Make chat usage visible while an answer runs, let the user change reasoning
effort, and provide clear Plan and Code modes beside the composer. Follow with
better context visibility, interruption recovery, and review of completed work.

Requested September 11, 2026. This is a proposed implementation plan; creating
this note does not implement features or authorize a release. Existing release
acceptance gates remain in [[AI Agent Next Steps Handoff]].

**Progress (2026-09-11):** C0, C3, and C4 are implemented in the app repo, per
the design at `docs/superpowers/specs/2026-09-11-reasoning-effort-and-plan-code-design.md`
and the audit in [[Provider Capability Audit — Effort and Plan Mode]]. Codex
reasoning effort is live (model-discovered options, per-turn override, no
reconnect). Plan/Code mode ships for all five providers, reusing each
adapter's existing native read-only mechanism; Codex, Claude, and Gemini are
verified available, Grok and OpenCode stay marked unverified pending a live
write-denial test. MCP vault-write tools are now filtered server-side under
read-only mode, closing the tool-exposure gap the audit found. C1/C2 (live
usage model/UI) and C5/C6 (recovery polish, integration evidence) remain
unstarted, as does effort support for Claude/Grok/OpenCode/Gemini.

This note refines usage and turn controls from [[AI Chat Conversation Experience]]
and [[AI Chat Conversation Experience Implementation Plan]]. Reuse their existing
conversation, context, retry, and approval features. Do not rebuild them from old
unchecked task lists. Related: [[AI Workspace UI and UX Plan]],
[[Agent Orchestration and Context Discipline]], [[Unified Knowledge System]],
[[Agent and Human Setup]], [[September 10 Improvements]].

## Verified local baseline

Source inspected September 11:

- `src/features/chat/codexChat.ts` handles
  `thread/tokenUsage/updated` and stores context used, limit, and remaining.
  Completed-turn token usage is currently derived from a context-usage delta.
  That delta is not a reliable general measure of tokens consumed by a turn,
  especially after compaction or context replacement.
- `src/features/chat/CodexChatPanel.tsx` already renders remaining context tokens.
  Extend this into a consistently visible composer status strip.
- `src/features/chat/chatProviderCapabilities.ts` declares five providers,
  permissions, images, resume, and reconnect behavior. It does not yet declare
  reasoning-effort choices, usage reporting semantics, or chat-mode support.
- Capability labels and help text need reconciliation with actual adapters;
  static declarations alone are not live provider certification.

External provider protocols and exact supported effort values are implementation
verification gates. This plan does not assert that all installed providers expose
the same controls or streaming usage.

## 1. Visible live token count — first delivery

Keep a compact usage strip immediately below the composer, visible during
streaming and after completion. Narrow layouts may wrap it without hiding Stop.

Example with illustrative reported values:

```text
[Code ▾]  [Model ▾]  [Effort: Medium ▾]  [Permissions ▾]
Message…
This turn: 1,240 tokens · Context: 18,420 / 128,000 · Running 12s
```

The strip opens a details popover with input, output, cached input, reasoning
tokens, conversation usage, context capacity, and last update time when reported.

### Measurement contract

- Treat turn consumption, cumulative conversation consumption, and current
  context occupancy as separate fields with separate labels.
- Normalize usage with provider, connection generation, thread, turn, timestamp,
  source, and semantics: delta or cumulative snapshot.
- Use authoritative usage events. Do not count streaming text chunks as tokens
  or derive turn consumption from context occupancy.
- Missing values remain unknown; distinguish unavailable, waiting for first
  report, partial report, final report, and restored historical snapshot.
- Providers that report only on completion show “Usage available after response.”
  A missing context limit must not hide known turn usage.
- Do not add cached or reasoning tokens twice when they are included in a
  provider total. Record field definitions for each adapter.
- Deduplicate events and reject stale connection/turn events. Reconcile late
  final usage with its original completed turn, never the newly active turn.
- Context can decrease after compaction; cumulative consumption must not be
  reduced by that decrease. Handle native resume, replay, retry, and branching
  without counting the same report twice.
- Render updates at most four times per second, with an immediate final update.
  This is a proposed UI budget, not a claim about provider reporting frequency.
- Show context warnings at proposed 80% and 95% occupancy only when the
  denominator is known. Offer context inspection and a fresh conversation.
- Omit unknown cost. Optional cost later requires provider-reported cost or
  verified, versioned pricing and compatible usage semantics.

Acceptance: reported usage changes visibly while a fixture streams, completion
settles the total, compaction changes only occupancy, and unsupported providers
never display a fabricated zero or an animated estimate.

## 2. Switchable reasoning effort

Add an Effort menu beside the model selector, showing the effective selection.
Use provider/model-discovered choices where available and verified adapter
metadata otherwise. Preserve provider-native values; do not invent a universal
Low/Medium/High mapping. “Provider default” is an explicit option.

- Persist the selection by workspace, provider, and model; record effective
  model and effort with each submitted turn.
- Validate restored choices after model or provider changes. If unsupported,
  show the fallback to provider default before sending.
- An idle change applies to the next turn. During a run, allow a pending choice
  labelled “Next message”; do not restart or silently alter the active turn.
- If an adapter requires reconnecting, preserve the draft and context, perform
  the transition at a safe boundary, and confirm effective settings before send.
- Display requested versus effective state until the adapter acknowledges it.
  Rejection leaves a recoverable error and the last confirmed state.
- Disable the control with a short explanation where effort is unavailable.
  Do not approximate an effort setting with prompt text.
- Explain effort as depth versus response time. Avoid promised token budgets,
  speedups, cost savings, or quality guarantees.

Acceptance: fixture requests receive the selected supported value; unsupported
values cannot be submitted; switching during streaming affects only a subsequent
turn; failures preserve the draft and confirmed settings.

## 3. Plan / Code mode

Mode expresses the task workflow. Permissions remain a separate visible control.

| Mode | Intended behavior | Completion action |
| --- | --- | --- |
| Plan | Inspect context, ask useful questions, produce a reviewable plan | Save plan or Implement plan |
| Code | Carry out the requested work under effective permissions | Review changes and validation |

### Plan behavior and enforcement

- Plan is read-only for agent actions: no file mutations, mutating shell
  commands, vault-write MCP tools, or worker execution.
- Enforce that boundary through verified provider/runtime restrictions and
  tool exposure. A system prompt or badge alone is insufficient.
- Where the installed adapter cannot enforce it, mark Plan unavailable and
  explain the limitation. Do not offer a deceptively protected fallback.
- Plan output includes objective, scope, affected areas, ordered tasks,
  dependencies, risks, and acceptance checks. Scale detail to the task.
- “Save plan” is a deliberate user action through the vault note workflow.
  Preview the destination and avoid overwriting an existing note silently.
- “Implement plan” prepares a Code turn containing the selected plan revision
  and relevant context. Show model, effort, and permissions before the user
  sends it. Do not launch workers merely by generating or importing a plan.
- Reuse the existing execution-plan importer when a plan needs orchestration;
  retain its draft review and dependency-aware launch gates.

### Switching and persistence

- Default new conversations to Code with the existing safe permission default.
  Restore a conversation's last confirmed mode when supported.
- Disable mode changes during an active turn or pending approval. The user can
  Stop, wait for cancellation acknowledgement, then switch.
- Switching into Plan must establish its effective read-only runtime before the
  next send. Switching back to Code must not silently grant Full access.
- Keep history, draft, and context through successful transitions. If a provider
  needs a fresh native session, disclose bounded replay and its context impact.
- Snapshot mode, model, effort, and effective permissions per turn so retries,
  branches, and resumed sessions do not silently inherit incompatible settings.

Acceptance: attempt mutations via repository tools, shell, vault MCP, and worker
launch in Plan fixtures; all must be unavailable or denied at the runtime
boundary. Code respects its selected permissions. A rejected transition sends
no turn under a misleading mode label.

## 4. Additional improvements

| Priority | Improvement | Acceptance |
| --- | --- | --- |
| P1 | Clear activity state: connecting, thinking, using tools, waiting for input, stopping, complete, failed | Stop remains reachable; stopped is shown only after acknowledgement |
| P1 | Context preview with source sizes and stale/missing markers | User can remove context before sending; token estimates, if added, are explicitly labelled |
| P1 | Recovery for failed sends and reconnects | Draft and attachments remain recoverable within existing storage policy; no duplicate automatic send |
| P1 | Per-turn configuration and usage summary | Expand a completed turn to inspect its actual mode/model/effort and reported usage |
| P2 | Completion summary with changed files and test outcomes | Reuse diff review; distinguish passed, failed, and not run from actual evidence |
| P2 | Context-pressure recovery | Offer a new conversation with a previewable handoff; native compaction only where supported |
| P2 | Keyboard and compact-window polish | Menus support keyboard, Escape, focus return, and visible labels; frequent usage updates do not flood screen readers |

Existing retry, transcript search, branching, context chips, and copy actions are
regression surfaces, not new deliverables. New providers, speculative billing,
automatic worker launch, and release publication are outside this plan.

## 5. Implementation sequence and ownership

Ownership denotes responsibilities for future implementation, not agents launched
by this note. Use one integrator for shared panel and reducer edits.

| Task | Owner / files | Depends on | Exit gate |
| --- | --- | --- | --- |
| C0: capability audit | Provider adapter maintainer; capability module, native bridges, protocol fixtures | None | Record installed versions, usage semantics, effort choices, Plan enforcement, reconnect requirements for all five providers |
| C1: usage model | Chat state maintainer; proposed `src/features/chat/chatUsage.ts`, reducer, conversation persistence | C0 | Delta/snapshot, duplicates, late events, compaction, resume and missing-field fixtures pass |
| C2: live usage UI | Chat UI maintainer; proposed `ChatUsage.tsx`, composer and panel integration | C1 | Always-visible strip and accessible narrow-layout smoke pass |
| C3: effort controls | Session maintainer; provider requests, settings persistence, composer | C0 | Effective and pending settings match actual request payloads |
| C4: Plan/Code | Runtime maintainer; native restrictions, MCP exposure, session transitions, composer | C0, C3 | Mutation denial and Plan-to-Code transition acceptance pass |
| C5: recovery and polish | Chat UI/session maintainer; existing context, activity, diff and recovery modules | C2, C3, C4 | P1 scenarios pass without changing existing save/approval behavior |
| C6: integration evidence | Integrator; tests and vault delivery notes | C5 | Automated checks plus recorded desktop/provider matrix |

Ship reviewable slices: live usage first, effort second, Plan/Code third, then
remaining polish. Reuse existing modules after locating their current paths;
avoid adding another large block of state management to CodexChatPanel.

C0 must classify each provider feature as verified, unsupported, or unverified.
An unavailable credential is unverified, not a passing cell. Prioritize a complete
Codex path while other providers display truthful capability fallbacks.

## 6. Validation and definition of done

Add focused behavioral tests for normalized usage, turn configuration, and mode
transitions. Extend existing session, provider, conversation, approval, and
repository-editor fixtures rather than duplicating them.

On the implemented revision, run from the app repo:

```sh
npm run check
node --experimental-strip-types --test scripts/*.test.mjs
cargo test --manifest-path src-tauri/Cargo.toml --lib
npm run build
git diff --check
```

Record desktop acceptance on macOS and Windows separately:

- Streaming usage, final-only usage, absent usage, and context compaction.
- Effort/model changes while idle, while streaming, and after reconnect.
- Plan mutation denial, explicit implementation handoff, and permission changes.
- Stop and late events, failed sends, retry, native resume, and bounded replay.
- Workspace/provider switches, old conversation migration, keyboard operation,
  compact window, and large transcript responsiveness.

Done means supported controls change actual runtime behavior, usage labels match
provider semantics, failures retain user work, and evidence identifies the exact
provider/model/runtime/platform tested. A successful build alone is insufficient.

Planning validation is documentation-only. Implementation tests and desktop
acceptance above remain pending until source changes are made.
