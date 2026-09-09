---
id: nodez-agent-orchestration-context-discipline
title: Agent Orchestration and Context Discipline
type: workflow
status: active
created: 2026-09-09
updated: 2026-09-09
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
