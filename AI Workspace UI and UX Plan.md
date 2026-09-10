---
id: nodez-ai-workspace-ui-ux-plan
title: AI Workspace UI and UX Plan
type: architecture
status: draft
created: 2026-09-09
updated: 2026-09-09
tags:
  - ai-chat
  - ui
  - ux
  - orchestration
  - workspace
---

# AI Workspace UI and UX Plan

This is the dedicated product-design page that follows [[AI Chat Conversation Experience]]. The chat page defines conversation behavior and runtime architecture; this page defines how the whole AI workspace should feel when chat, notes, code, graph context, approvals, and agent orchestration appear together.

## Product intent

The AI workspace should make the agent's scope, progress, authority, and evidence legible at a glance. A user should know which vault and repository are active, what context will be sent, whether the graph is current, what the agent is doing, and what requires approval. The interface should remain calm during streaming and become more explicit only when a decision or failure needs attention.

The workspace is a local-first tool. Navigation and drafting remain useful without a provider connection. Agent actions never imply that a source file changed until an explicit editor Save succeeds.

## Information architecture

The primary workspace has four coordinated surfaces:

1. **Workspace rail** — vault, repository, recent notes, search, graph, setup, and settings.
2. **Main content** — Markdown note, repository editor, graph view, or focused setup page.
3. **AI chat panel** — conversation history, messages, activity, composer, context chips, and approvals.
4. **Evidence drawer** — optional detail view for graph relationships, source citations, diffs, usage, and task status.

On narrow windows, the rail and evidence drawer become temporary panels. Chat remains reachable without hiding the active note or editor context.

## Placement after AI chat

Keep this page directly after [[AI Chat Conversation Experience]] in the docs navigation and link both pages to each other. The chat page remains the canonical reference for sessions, messages, providers, usage, approvals, and persistence. This page owns cross-surface composition, responsive behavior, visual hierarchy, accessibility, and orchestration status.

Do not duplicate provider protocol decisions here. Reference the chat page for those contracts and the orchestration page for host-agent worker policy.

## Core user journeys

### Understand before sending

The user sees the active vault and repository in the workspace header. The composer shows removable chips for the current note, file, selection, pinned context, and graph context. Each chip opens a bounded preview with freshness and provenance. Sending is disabled only when a required context item is missing or invalid; otherwise stale context is clearly marked and can be removed.

### Follow work while the agent runs

A turn has one stable visual position. Streaming text grows in place, tool activity is grouped beneath the turn, and the Stop control stays in the same location as Send. Running work is visible without making the entire panel flash or jump. Late events from a previous session never appear in the active transcript.

### Make an approval decision

Approval cards explain the requested action, affected path, provider, permission mode, and available choices. Allow once, allow similar this session, and Deny are visibly distinct. Similar-command rules show their session scope and expiry. Read-only mode never presents a misleading write affordance.

### Inspect and apply a proposed edit

A proposal opens in the repository editor with changed files, validation state, and revision information. Apply, Save, Undo, Reload, and Cancel have separate meanings. The UI distinguishes an in-memory proposal from a saved repository change. Conflicts identify the changed buffer and preserve the proposal for review.

### Recover from low context

When reported remaining context reaches the existing warning threshold, the composer shows a compact warning and offers a structured handoff. A successful handoff opens a new continued conversation while retaining the original history. A failed or empty handoff leaves the current conversation intact.

### Trace an answer to evidence

Graph context and citations show node, relationship, provenance, confidence, source path, and line or heading where available. Selecting a citation opens the exact target in the note or editor, with unsaved-navigation confirmation when needed. Missing targets show an actionable error.

## Orchestration surface

The host orchestrator should be represented as a lightweight task status area, not as a second chat transcript. It shows:

- task goal and current phase;
- assigned worker and owned scope;
- queued, running, awaiting review, accepted, or blocked state;
- latest verification result;
- links to diffs, logs, and vault handoffs;
- blockers requiring a human decision.

Workers do not need to expose internal reasoning. The UI displays decisions, progress, evidence, and next actions. Parallel workers are grouped by task and visually separated from the coordinator's integration state.

A review gate requires explicit evidence. “Done” from a worker means ready for review; “accepted” appears only after the coordinator or refuter verifies the actual diff and checks.

## Visual hierarchy

Use a quiet base surface for notes and code, with stronger contrast reserved for active turns, approvals, errors, and unsaved state. Keep provider, model, permission mode, and workspace identity close to the connection control. Keep destructive or authority-changing actions behind explicit labels.

Prefer stable regions over modal stacking:

- connection and permission controls in the chat header;
- context chips in the composer;
- approval cards in the transcript;
- source and graph evidence in the drawer;
- task orchestration in a compact status panel.

Avoid exposing raw JSON, opaque event names, internal session IDs, or provider-specific jargon when a human-readable label is available.

## Responsive and accessibility requirements

- Preserve keyboard navigation for chat history, context picker, message actions, approvals, graph citations, and task status.
- Keep focus after sending, closing a popover, applying a proposal, or returning from a citation.
- Announce streaming completion, approval requests, failures, and successful saves to assistive technology.
- Ensure color is not the only signal for stale, running, blocked, or unsaved states.
- Support reduced-motion preferences; streaming and panel transitions must not obscure content.
- On small screens, expose one focused panel at a time with a clear return path and no loss of draft text.
- Maintain visible focus rings and adequate hit targets for Stop, approval, Save, Undo, and context removal.

## State model

The UI should model these states independently:

- workspace: no vault, vault open, repository attached, repository unavailable;
- connection: disconnected, connecting, ready, streaming, stopping, failed;
- graph: building, current, stale, error;
- editor: clean, dirty, proposal pending, conflict, saving, saved, failed;
- orchestration: queued, running, review, accepted, blocked;
- context: current, stale, missing, pinned, removed.

Do not collapse these into one global loading flag. A connected provider can coexist with a stale graph; a saved chat can coexist with a disconnected provider; an in-memory editor proposal can coexist with an unchanged repository.

## Delivery sequence

1. Audit the existing chat, editor, graph, and setup surfaces against the state model.
2. Establish shared visual tokens and status language without restructuring runtime modules.
3. Add the workspace header and compact evidence drawer.
4. Refine context chips, graph previews, citations, approvals, and editor proposal states.
5. Add the orchestration status panel backed by durable handoff references.
6. Run keyboard, screen-reader, reduced-motion, narrow-window, and interrupted-session checks.
7. Record screenshots or short evidence notes in the vault for accepted states.

Each slice must preserve explicit repository saves, vault-only MCP writes, provider permission meanings, and the current chat session boundaries.

## Acceptance criteria

- A user can identify active vault, repository, provider, model, and permission mode without opening a settings page.
- Before sending, context is visible, removable, bounded, and marked current, stale, or missing.
- During streaming, Send becomes Stop in place and old-session events cannot alter the active turn.
- Approval cards state the action and scope and never imply a write succeeded before confirmation.
- Proposed edits clearly separate Apply, Save, Undo, Reload, and conflict recovery.
- Graph and file citations open the recorded evidence target or report a precise missing-target error.
- Orchestration status distinguishes worker completion from reviewed acceptance.
- The layout remains usable at narrow widths and with keyboard navigation and reduced motion.
- No UI copy claims native OTA, automatic source writes, or inferred relationships as extracted facts.

## Related

- [[AI Chat Conversation Experience]]
- [[Agent Orchestration and Context Discipline]]
- [[AI Agent Next Steps Handoff]]
- [[Code Editor and AI Chat Plan]]
- [[Unified Knowledge System]]
- [[Command Palette and Agent Surface]]

## Orchestration UI: assign an orchestrator and workers

The orchestration UI should make a multi-agent run understandable without turning Nodez into an opaque autonomous system. The user explicitly starts a run, chooses the orchestrator, assigns workers, and controls the review gates.

### Run structure

A run contains one orchestrator and zero or more workers:

- **Orchestrator** — owns decomposition, task sequencing, shared-file coordination, integration, and the final accept or reject decision.
- **Worker** — owns one bounded task and a declared file or subsystem scope.
- **Refuter** — independently reviews a worker result and its evidence before acceptance. A refuter may be a separate provider session or a coordinator-assigned review task.

The initial run setup should require an explicit orchestrator. Workers can then be added one at a time or from a small preset. The first release should cap active workers at the host's available slots and default to two concurrent workers for safety.

### Assignment flow

1. Select **New orchestration run** from the command palette or AI workspace.
2. Enter the goal and choose the active vault and repository context.
3. Select the orchestrator provider, model, and permission mode.
4. Review the generated task breakdown and edit task boundaries.
5. Add workers by selecting a provider and model, including OpenCode where it is installed and ready.
6. Assign each worker a task, owned paths, acceptance checks, and focused test commands.
7. Confirm the run. The UI records the assignment before any worker session starts.
8. Start workers individually or start all independent tasks together.

The confirmation screen must show the complete assignment table and identify overlapping paths before launch. Overlap requires the user to resolve ownership or mark the task read-only.

### Assignment card

Each worker card displays:

- provider and model;
- permission mode;
- task goal;
- owned files and directories;
- prohibited paths;
- base revision;
- dependencies and blocked-by tasks;
- required checks;
- current state;
- latest report and evidence links.

OpenCode is presented as a provider choice with its actual capabilities. The card must not imply that Nodez can select arbitrary internal OpenCode subagents unless that capability is explicitly exposed by the adapter. An OpenCode worker is one bounded ACP session in the first version.

### Run board

Use a board or grouped timeline with these states:

- Draft
- Ready
- Running
- Waiting on dependency
- Awaiting review
- Changes requested
- Accepted
- Blocked
- Cancelled

The orchestrator lane remains visually separate from worker lanes. A worker moving to Awaiting review does not advance the run automatically. The orchestrator must inspect the report, and the refuter must record a pass or a concrete defect.

A compact header shows the run goal, active workspace, base revision, active worker count, and the next gate. A detail drawer shows the assignment envelope, event timeline, changed files, test results, and handoff note without exposing hidden reasoning.

### Review and integration flow

When a worker finishes, Nodez freezes its assignment metadata and captures:

- changed files;
- behavior before and after;
- exact tests and results;
- live scenarios, if any;
- artifacts and evidence paths;
- remaining risks;
- shared-file changes requested.

The orchestrator can accept, request changes, reassign review, or mark the task blocked. Acceptance requires an actual diff and verification evidence. The UI should make it impossible to confuse a worker's completion message with an accepted change.

After all required tasks are accepted, the orchestrator opens an integration gate. The gate lists overlapping changes, shared files, focused checks, full checks, and unresolved risks. Integration remains an explicit coordinator action and never happens merely because all workers report success.

### Context and permission boundaries

The run inherits the active workspace identity but each worker receives a bounded context manifest. The manifest includes selected notes, graph results, source paths, and task instructions. It does not silently include another workspace, another provider's private session, secrets, raw event history, or unselected transcript content.

Permission mode is visible per agent and fixed for the session. Read-only workers cannot be presented with a Save or Apply action. Ask mode shows approval requests in the worker lane. Full access requires an explicit selection and a visible confirmation before launch.

MCP writes remain vault-only. Repository edits continue to use the separate editor capability and explicit Save. The orchestration UI must label a vault-note update, an in-memory proposal, and a saved source change as different outcomes.

### Failure, cancellation, and recovery

Stopping a worker marks it stopping, then cancelled or failed after the process exits. Late events are discarded using the existing workspace, provider, session, thread, and turn identity checks. Cancelling the orchestrator pauses scheduling and leaves completed worker evidence intact.

If a worker disconnects, the card offers Resume when the provider supports native resume, Restart with the same assignment, or Mark blocked. Restart creates a new session identity and never merges late output into the old transcript.

A run can be reopened from a persisted handoff. Persist assignment metadata, statuses, reports, evidence paths, and decisions. Do not persist secrets, approval rules, raw provider events, or image payloads.

### First implementation slice

Build the orchestration UI in this order:

1. Static run board with one orchestrator and manually added workers.
2. Assignment envelope editor with path-overlap validation.
3. Provider/model/permission selection using existing capability records.
4. Worker status timeline and compact report capture.
5. Refuter review and explicit acceptance gate.
6. Integration checklist and links to diffs, tests, and vault handoffs.
7. Scheduling, cancellation, restart, and persistence hardening.

The first slice may use mocked worker lifecycle events to validate the interaction model. It must not claim that mocked events prove real multi-agent execution. Connect the board to live provider sessions only after assignment, identity scoping, cancellation, and late-event rejection are covered by tests.

### Orchestration UI acceptance criteria

- A user can select exactly one orchestrator and assign at least two distinct workers.
- Every worker has visible provider, model, permission mode, goal, owned paths, base revision, and acceptance checks.
- Path overlap is detected before launch and requires an explicit resolution.
- Independent workers can run concurrently within the configured cap.
- Dependency-blocked tasks do not start prematurely.
- Completion, review, acceptance, rejection, cancellation, and blocked states are distinct.
- A refuter can review the actual diff and exact test evidence.
- The coordinator must explicitly approve integration.
- OpenCode can be assigned as a bounded worker session without implying unsupported internal subagent control.
- Workspace, provider, session, thread, and turn boundaries prevent late output contamination.
- Reopening a run restores assignments and evidence without restoring secrets or raw provider events.
- Keyboard navigation, reduced motion, narrow-window layouts, and screen-reader announcements work across setup, assignment, review, and integration gates.

### Provider-aware worker selection

The assignment dialog should list every installed and ready provider that Nodez supports: Codex, Claude, Grok, OpenCode, and Gemini. Availability comes from the existing readiness surface and must be shown as a concrete state such as Ready, Not installed, Authentication required, Timed out, or Probe failed. A provider that is unavailable can remain visible for setup, but cannot be launched as a worker.

The worker card should derive its controls from the provider capability record:

| Provider | Worker selection behavior |
| --- | --- |
| Codex | Read-only, Ask, or Full access; approvals and images available; native resume supported when advertised |
| Claude | Read-only or Full access; no mid-turn approval control; model and permission are fixed at connection; images available |
| Grok | Read-only, Ask, or Full access; approvals available; images unavailable; reconnect on model or permission change |
| OpenCode | Read-only, Ask, or Full access; approvals and images available; one bounded ACP worker session in the initial version |
| Gemini | Read-only, Ask, or Full access when the installed CLI supports the required Plan behavior; approvals and images available; API-key readiness is explicit |

The table is a product-facing summary only; the capability registry remains the source of truth. The UI must hide unsupported controls rather than rendering disabled actions that suggest the provider can perform them. For example, Claude does not receive an Allow once card, and Grok does not receive an image attachment affordance.

A worker assignment records provider, model, permission mode, and capability snapshot at launch. If readiness changes afterward, the running assignment keeps its identity and reports the failure or reconnect requirement; it does not silently switch providers or permission modes.

The orchestrator may mix providers in one run. A typical run could use Codex for integration, Claude for a read-only review, Grok for an independent implementation, and Gemini for a second review when all are ready. Cross-provider context is bounded to the assignment manifest. Conversation carry is opt-in and never copies private provider session state automatically.

The setup dialog should provide a clear fallback path: open provider settings, show the exact missing prerequisite, or continue with another ready provider. A missing Gemini key, unavailable Grok CLI, or missing Claude authentication blocks only that worker assignment; it does not block unrelated workers or the coordinator.

### Orchestrator-led model assignment

The orchestration setup must make the orchestrator the decision point for both work decomposition and model routing. The user chooses or confirms the orchestrator, then the orchestrator proposes worker tasks, provider assignments, model selections, capability floors, and dependencies. The user can edit or approve the proposal before launch.

Each worker row shows:

- task and owned paths;
- provider and selected model;
- capability floor;
- why that model was selected;
- permission mode;
- estimated context and cost when reliable;
- escalation option if the model cannot complete the task.

The default is the lowest capable model for each provider. A low-cost model handles scouting and routine bounded changes; stronger models are reserved for difficult debugging, independent refutation, security or release review, architecture, and final integration. The UI should make this visible without ranking providers as universally better or worse.

The orchestrator may assign different providers to different tasks. It must account for actual capability records: Claude has no mid-turn approvals, Grok has no image input, Gemini read-only depends on Plan support, and OpenCode is initially one bounded ACP session. If no model satisfies a task's floor, the task remains unassigned or blocked with a concrete reason.

Model escalation requires an explicit event in the run timeline. The event states the failure or capability gap, previous model, new model, and revised acceptance check. Switching models never changes file ownership, workspace identity, permission semantics, or the evidence required.

### Provider pool control

The run setup includes a provider-pool selector with three mutually clear modes:

1. **All available** — automatically include every ready provider whose advertised capabilities satisfy the task.
2. **One provider** — use only the selected provider for the orchestrator and workers.
3. **Custom combination** — choose the exact providers the orchestrator may use.

The selector shows readiness, authentication, model availability, and capability warnings before launch. “All available” never means “force every provider to run”; it means the orchestrator may use any eligible provider and may leave one unused when no task fits it.

The assignment preview explains the resulting distribution. For example, Codex may orchestrate, Claude may perform a read-only review, Grok may handle a routine implementation, and Gemini may perform an independent review. OpenCode may be included as a bounded ACP worker when its readiness check passes.

Users can lock a task to a provider, allow the orchestrator to choose within the pool, or leave it unassigned until a required provider becomes ready. A provider lock does not override the model capability floor. If the locked provider cannot satisfy the floor, the task is blocked with a setup or escalation action.

The pool choice is persisted with the run so that reopening it does not silently introduce a new provider. Newly available providers are proposed explicitly for queued work and require confirmation before launch.
