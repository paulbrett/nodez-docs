---
id: nodez-ai-chat-follow-on-phases
title: AI Chat Follow-on Phases
type: roadmap
status: draft
created: 2026-09-09
updated: 2026-09-09
tags:
  - ai-chat
  - roadmap
---

# AI Chat Follow-on Phases

Related: [[AI Chat Conversation Experience]], [[AI Workspace Delivery]].

1. **Conversations** — naming, recent-session resume, search, pin/archive,
   branching, and handoff export are implemented.
2. **Context trust** — per-message labels, stale warnings, pinned project context,
   and automatic summarize-and-continue are implemented.
3. **Multi-file changes** — reviewed change sets, per-file diffs, explicit
   apply/discard controls, save/conflict handling, and an approval audit trail.
4. **Provider operations** — readiness diagnostics, accurate provider-reported
   usage, and full provider/platform acceptance.

Next delivery order: finish provider/platform acceptance, then release validation.

## Reviewed change sets — implemented 2026-09-09

Keep the first slice deliberately small:

1. Normalize provider file-change events into one change-set model keyed by
   workspace, conversation, turn, and base file revision.
2. Show a compact summary with changed-file count and additions/deletions, then
   open per-file diffs on demand.
3. Allow Apply or Discard per file plus Apply all. Applying updates editor
   buffers; ordinary explicit Save remains the disk-write boundary.
4. Reject stale revisions and files outside the authorized editor root. Preserve
   accepted and rejected decisions as visible transcript activity.

Acceptance: a two-file proposal can be reviewed independently, a stale or
out-of-root change cannot modify a buffer, Apply all is one explicit action, and
provider/workspace switches cannot deliver a late change set into the new chat.

## Provider readiness — implementation landed 2026-09-09

Add a provider health view covering CLI presence/version, credential status,
ACP/App Server handshake, advertised modes/models, and a minimal prompt. Keep
secrets hidden and report usage only when the provider supplies a valid context
limit. The native readiness command and UI are implemented. Complete live macOS
checks for Codex, Claude, Grok, OpenCode, and Gemini, then run the same acceptance
matrix on Windows before release packaging.
