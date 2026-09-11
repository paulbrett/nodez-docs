---
id: nodez-agent-orchestration-context-discipline
title: Agent Orchestration and Context Discipline
type: workflow
status: active
created: 2026-09-09
updated: 2026-09-10
tags:
  - agents
  - orchestration
  - context
  - workflow
  - handoff
---

# Agent Orchestration and Context Discipline

Practical pattern for using multiple AI agents without wasting the strongest model on work cheaper workers can do, and without flooding the orchestrator context with large code or research dumps.

This is **host-agent workflow guidance**, not a new Nodez MCP/product API. Keep the layering from [[Agent Skills and Surfaces]]: Nodez exposes tools through MCP; orchestration strategy belongs to the host/coding-agent layer and can use vault notes as durable handoff context.

## Core principle

**The orchestrator is the coordinator, not the default worker.**

Use the strongest reasoning model mainly to:

- plan and decompose work
- write precise specs and acceptance criteria
- decide architecture and tradeoffs
- dispatch narrowly scoped agents
- read short reports
- integrate results and make final judgment calls

Do not spend the orchestrator's context on huge code reads, bulk refactors, long documentation sweeps, or repetitive work that a cheaper worker can handle safely.

Small tasks are the exception: a one-line fix, a single grep, or a trivial local edit can be done directly when spawning another agent would cost more than the work itself.

## Recommended roles

| Role | Responsibility | Expected report |
| --- | --- | --- |
| **Orchestrator** | Plan, scope, specify, dispatch, integrate, decide | Final decision and next action |
| **Scout** | Find files, symbols, call sites, references, affected areas | Locations and concise findings; no whole-file dumps |
| **Researcher** | Read docs/source and verify facts | Short factual summary; mark uncertainty/unverified claims |
| **Builder** | Implement from a clear spec and run tests | What changed, files touched, tests/results |
| **Refuter / Reviewer** | Independently review the builder's diff and rerun verification | Defects, risks, failed checks, or explicit pass evidence |
| **Debugger** | Hard root-cause investigation only | Root cause, evidence, minimal repair path |

Use stronger models selectively for difficult debugging or high-consequence review rather than by default for every task.

## Delegation contract

Every sub-agent should receive a strict task envelope:

1. **Specific goal** — one clear outcome.
2. **Exact scope** — files, directories, symbols, URLs, or subsystem boundaries.
3. **Allowed changes** — what it may edit and what must remain read-only.
4. **Verification required** — tests, commands, diff checks, or facts it must validate.
5. **Required output format** — findings, patch summary, test report, etc.
6. **Short output limit** — enough to act on, not a transcript of the work.
7. **Known context** — include established facts so the worker does not spend time rediscovering them.

The orchestrator should ask for **locations and conclusions**, not giant code dumps.

## Context-budget rules

- Keep orchestrator replies and worker reports short unless deeper detail is required.
- If a worker produces a large amount of useful information, write it to a **scratch/handoff file or note** and return only a compact summary plus the path/reference.
- Persist accepted decisions, progress, constraints, and unresolved questions in durable handoff docs so the next session can resume from files instead of reconstructing the full chat context.
- Batch related fixes so the same large files are not repeatedly reread by multiple agents.
- Pass established facts forward between agents instead of paying to rediscover them.
- Do not treat a worker's “done” claim as verification evidence.

Nodez is a natural place for the durable layer: accepted decisions and handoffs can live in the vault, while source changes remain in the code repository.

## Default coding loop

```text
Orchestrator -> Builder -> Refuter -> Orchestrator
```

1. **Orchestrator** creates the spec and acceptance criteria.
2. **Builder** makes the change and runs its checks.
3. **Refuter** reviews the actual diff and reruns verification independently.
4. **Orchestrator** integrates the evidence, accepts/rejects the result, and decides the next step.

For discovery-heavy work, insert Scout/Researcher before Builder. Use Debugger only when the failure needs deeper root-cause work.

## Parallelism rules

Read-only tasks can run in parallel when they do not depend on each other's output:

- research
- codebase scouting
- independent review
- documentation/source verification

Avoid multiple agents editing the same files at the same time. Prefer one active writer per overlapping file set, then independent review afterward.

**Builders build; refuters verify.** Keeping those responsibilities separate reduces confirmation bias and makes test evidence more trustworthy.

## Agent-count and escalation rules

- Do not spawn an agent for every tiny task.
- Keep larger multi-agent workflows opt-in or intentionally triggered.
- Cap the number of workers for broad tasks.
- Stop an agent that leaves scope instead of letting it continue consuming time/context.
- Escalate to a stronger model only when the task's difficulty or risk justifies it.

## Handoff report template

A worker's normal return should be compact:

```text
Goal: <what was requested>
Status: done | blocked | needs review
Findings/changes:
- <short item>
- <short item>
Verification:
- <test/check and result>
Files/references:
- <paths only>
Large output:
- <scratch/handoff path, if any>
Risks/unverified:
- <only what still matters>
```

The report is an interface between agents, not a replay of the worker's internal process.

## Nodez application

This pattern reinforces existing Nodez architecture:

- **Vault** = durable intent, decisions, specs, handoffs, research summaries
- **Repository** = implementation truth and diffs
- **Graph** = discovery/relationship context
- **MCP** = bounded tools for agents to find/read/write vault knowledge
- **Host agent** = orchestration, model selection, parallelism, and worker lifecycle

Do not move host orchestration policy into Nodez's MCP contract unless Nodez later becomes responsible for spawning/managing workers itself.

## Related

- [[Agent Skills and Surfaces]]
- [[Agent and Human Setup]]
- [[AI Agent Next Steps Handoff]]
- [[Unified Knowledge System]]

## Execution plan — release readiness, attachments, and graph trust

This plan turns the orchestration pattern into a repeatable delivery sequence for Nodez. The host agent coordinates; workers operate inside explicit ownership boundaries; the vault records accepted intent and evidence.

### Phase 0 — Baseline and acceptance matrix

The coordinator records the branch, commit, dirty and untracked files, inherited changes, package versions, available worker slots, and the current graph freshness state. It creates or updates a release acceptance matrix in the vault. Existing failures are separated from new failures, and fixture evidence is kept separate from live provider and packaged-app evidence.

No worker starts until its base revision, owned files, prohibited files, required checks, and inherited changes are explicit. Shared integration files such as App.tsx, shared styles, and Tauri routing have one active writer.

### Phase 1 — Parallel release-readiness work

Provider work uses deterministic fixtures for streaming, approvals, cancellation, malformed events, late events, model lists, provider switches, workspace switches, and conversation carry. Live provider checks follow only after fixture behavior is verified.

Readiness and credential work bounds probes, reuses runtime launcher resolution, distinguishes missing executables from timeouts and probe failures, and verifies that failed credential writes preserve the previous readable store. Secrets never appear in logs or reports.

Editor and search work exercises asynchronous UI races: changed or closed buffers, stale proposals, repeated Apply, root and permission changes, external edits, save conflicts, reload, undo, preview searches, Unicode, duplicate titles, keyboard navigation, and a 1,000-note fixture.

Update work adds a pure SemVer comparator, tests release-candidate precedence and malformed feeds, validates installer targets and URLs, and documents the current download-based update behavior. It does not enable native OTA as an incidental fix.

### Phase 2 — Review and integration gate

Each builder returns a compact report and its actual diff is reviewed by an independent refuter. The coordinator integrates only reviewed changes, resolves shared-file conflicts, and runs the focused suites followed by the full repository checks. A passing unit test is evidence for that behavior; it is not evidence of desktop packaging or live provider acceptance.

### Phase 3 — Platform certification

The release worker builds from the reviewed revision and records toolchains, targets, timestamps, paths, and checksums. macOS and Windows smoke tests use disposable profiles and scratch vaults. Provider modes, Stop, approvals, saves, conflicts, non-ASCII paths, MCP launcher resolution, and the N−1 to N download flow are tested on the platform where they are claimed.

If publication authorization is absent, the process ends with a complete release report and does not tag, publish, or dispatch a public workflow.

### Phase 4 — Attachments feature slice

After release readiness, the coordinator records the attachment contract, then assigns storage and editor integration separately. Imports remain vault-contained, collision-safe, bounded, portable after vault relocation, and tied to the originating note and editor transaction. Successful references are inserted as one undoable operation; partial failures are visible and recoverable.

### Phase 5 — Graph freshness and source navigation

The graph worker traces note writes, watchers, pulls, commits, MCP rebuilds, and worker persistence. It adds workspace generation guards for out-of-order builds, verifies that source metadata and HEAD survive vault-only rebuilds, and preserves the commit-only branch|HEAD indexing contract.

The UI then distinguishes building, current, stale, and error states; reports the last successful persistence; offers retry after real failures; and opens citations at recorded files, lines, or headings with provenance. Missing targets fail visibly instead of opening an unrelated file.

### Worker report required at every gate

Each report names the task and owner, base revision and inherited edits, outcome, changed files, before/after behavior, exact test results, live scenarios, evidence paths, blockers, shared-file needs, authorized commits, and the next recommended task. Large logs belong in scratch files or vault handoffs; the chat contains conclusions and references only.

### Stop and escalation rules

Do not spawn a worker for a trivial edit. Do not run several writers against the same files. Stop workers that leave scope. Escalate to a stronger model for difficult root-cause debugging or high-consequence review, not by default. Do not expand this delivery sequence into new providers, autonomous agents, fuzzy search, collaboration services, an LSP rewrite, or graph decoration without a separate decision and acceptance record.

## Model routing and task assignment policy

The orchestrator is responsible for deciding what work belongs to each worker and which model should perform it. Workers do not self-expand their scope or choose a stronger model merely because it is available.

For every task, the orchestrator records a capability floor: discovery, routine implementation, test repair, protocol reasoning, security-sensitive review, architecture, or release judgment. It then selects the lowest-cost model available from the chosen provider that meets that floor. “Lowest” means the least expensive or least resource-intensive model in that provider's advertised catalog, while still meeting the required context, tool, reasoning, and modality capabilities.

Use this default routing:

| Task class | Default model policy |
| --- | --- |
| Scout, file locating, simple inventory | Lowest capable model |
| Routine test additions and bounded implementation | Lowest capable coding model |
| Focused debugging or multi-file coordination | Mid-tier reasoning model when the lowest model fails the capability floor |
| Refutation, security, release, or data-ownership review | Stronger reasoning model selected explicitly by the orchestrator |
| Architecture decomposition and final acceptance | Strongest available orchestrator model |

Provider choice and model choice are separate decisions. The orchestrator may assign Claude, Grok, Gemini, OpenCode, or Codex according to readiness, capability, cost, and task fit. A provider with no model meeting the floor is marked unavailable for that task; Nodez must not silently substitute a weaker model or switch providers.

Each assignment records provider, model, capability floor, reason for selection, permission mode, and expected evidence. The UI should show a short explanation such as “lowest capable model for TypeScript test repair” or “strong review model required for release semantics.”

Escalation is evidence-driven. A worker may request escalation when it encounters a capability mismatch, repeated test failure, insufficient context, or a high-risk decision. The orchestrator reviews the request, may move the task to a stronger model, and records why. Escalation does not broaden the worker's file ownership.

The orchestrator should prefer a lower model for independent parallel work and reserve the strongest model for decomposition, difficult diagnosis, review disputes, integration, and final judgment. Cost and latency are visible run metrics, but they never override permission boundaries or acceptance requirements.

## Provider pool selection

Before decomposing work, the orchestrator receives a provider-pool policy chosen by the user:

- **All available providers** — consider every provider that passes readiness and has a model meeting the task capability floor.
- **Single provider** — constrain all worker assignments to one selected provider while preserving per-task model routing.
- **Custom combination** — select two or more providers and let the orchestrator distribute tasks among that set.

“All available” means ready providers in the current workspace, not every provider known to the application. Providers that are missing, unauthenticated, timed out, or capability-incompatible are excluded with a visible reason. Their absence must not silently reduce a task below its capability floor.

The orchestrator uses the selected pool to balance capability, cost, latency, modality, and independent-review value. It should avoid assigning the same provider and model to both builder and refuter when an independent provider is available and the task benefits from independent review. This is a routing preference, not a requirement when only one provider is ready.

The pool policy is captured in the run record and shown in the assignment summary. Changing the pool after workers start creates a new scheduling decision and does not mutate existing session identities. A user may add a newly ready provider only to queued or unassigned tasks; running and accepted tasks remain unchanged.

## Guided orchestration planning — September 10

The in-app planning surface now follows **Goal → Team → Review**. The goal step
asks for the desired outcome and offers **Suggest team with AI**. The existing
AI Chat connection handles this planning request. A disconnected or busy chat
keeps the request queued; the user can cancel it before it is sent. Planning
preserves unsent text and image attachments in the chat composer.

AI suggests a small task breakdown, scopes, and acceptance checks. Nodez routes
those tasks to eligible models using the declared capability floors. Simple work
can use one worker. Provider selection, coordinator settings, concurrency, and
per-worker overrides live under **Customize**. The initial provider pool contains
ready providers, and the coordinator defaults to a model meeting the architecture
floor when one is available.

A valid response opens the Team step automatically. New suggestions append draft
assignments without overwriting existing work. Subsequent requests include the
existing task list and ask for missing tasks only. Applying a seed is idempotent
under React StrictMode. Malformed task entries reject the response as a whole;
missing checks remain visible for the human to complete.

The Review step shows assignments, scopes, checks, and permissions. Activity,
simulation controls, and integration evidence are optional disclosures. A ready
task can start an isolated provider session from its assignment card; completion
moves it to review, while failures remain restartable. Reviewing a plan itself
does not launch workers or save source files. Nodez MCP still cannot mutate the
indexed source root.

Validation: the orchestration unit suite covers malformed replies, preservation
of suggested checks, eligible model routing, and draft worker identities. Browser
fixtures exercise guided navigation, customization, additive suggestions, restored
runs, narrow layouts, StrictMode, queued planning, cancellation, draft preservation,
valid replies, malformed replies, and failed requests. Provider replies in these
browser checks are simulated; they do not certify live provider execution.

## Task branching — September 10

Failed or changes-requested tasks can be branched from their assignment card.
The original task and its evidence remain unchanged. The branch receives a new
task/session identity, starts as a clean draft, and preserves the original
provider, model, permission, scope, and checks. Branch lineage is stored in the
run snapshot and shown in the Team step.

## Orchestrator planning and synthesis — September 10

Nodez uses a hybrid orchestrator-worker architecture. The coordinator is an
explicit assignment with its own provider, model, and permission mode. It may
ask the connected AI to return a fenced JSON plan; Nodez validates that plan,
routes each task to a model meeting its capability floor, checks path ownership
and dependencies, and schedules independent workers in parallel. Local
heuristics remain available for small, predictable suggestions.

Worker sessions return structured evidence rather than raw transcripts. A
read-only refuter can inspect each completed task and record a pass or defect.
Verified accepted reports can be inspected as branch diffs and merged only by
an explicit human action. The parent checkout must be clean and the merge is
left uncommitted.

After workers settle, **Copy final synthesis brief** creates a bounded handoff
for the orchestrator containing the goal, accepted reports, checks, reviews,
risks, and run gates. The orchestrator's final response must distinguish
completed evidence from unresolved risks and state whether human merge approval
is still required. Future work may add a direct synthesis turn and opt-in
reusable workflow notes; automatic learning is not enabled by default.

The Team step presents capability floors as role presets: Researcher,
Implementer, Test engineer, Systems engineer, Security reviewer, Architect,
and Release judge. These labels make the AI's routing legible while the stable
floor remains the validation and model-selection contract.

The Review step also offers **Ask orchestrator to synthesize**. This sends the
bounded evidence brief directly into the connected AI chat and returns the
coordinator's final outcome, evidence summary, risks, and merge requirement.
The copy action remains available when the user wants to inspect or edit the
brief first.

Review now has a direct **Start orchestration** action. It queues all draft
assignments, returns to the Team step, and auto-starts every dependency-ready
worker within the concurrency limit. The coordinator chooses read-only mode
for inspection work and preserves write access only for scoped implementation
tasks, so users do not need to configure each permission before running.

Pending refuter reviews remain scheduled after restoring a saved run. If the
user opens Finish while a verdict is pending, Nodez returns to Activity so the
review runner remains mounted and can start; Finish also exposes a **Start
review** fallback instead of leaving the task at an unexplained gate.

Accepted runs can also export a reusable workflow template. This is explicit
and opt-in; it contains task roles, scopes, and checks, while excluding chat
transcripts and execution history.

## Execution-plan import and worker lifecycle — September 10

The Goal step can import a structured execution-plan note from the active vault.
The user selects one plan and one run, reviews its parsed tasks, and applies them
as drafts. Import preserves task keys, dependency edges, capability floors,
permissions, owned and prohibited paths, acceptance checks, shared invariants,
source-note identity, and the selected run. A recorded base-revision mismatch is
shown before import. Duplicate imports are blocked, and importing never launches
a worker.

**Start orchestration** now queues draft assignments and enables dependency-aware
automatic scheduling within the configured concurrency cap. Active runner
components stay mounted through provider connection and execution. Connection
failures settle as Failed; Stop interrupts and disconnects before settling as
Cancelled. Draft and settled tasks can be removed, and removal clears their IDs
from remaining dependency lists. Active tasks must be stopped first.

Review begins without a preselected pass verdict. A real provider reply that does
not match the structured report contract receives one corrective retry and
remains explicitly unverified if it still cannot be parsed. It is not mislabeled
as a simulation. Refuters run read-only and use a stronger model from the same
provider when one is declared.

Focused orchestration and plan-import coverage is part of the complete JavaScript
suite. Live multi-provider execution and packaged-app orchestration acceptance
remain separate gates.
