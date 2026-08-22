---
id: diamante-agent-skills-and-surfaces
title: Agent Skills and Surfaces
type: architecture
status: active
created: 2026-08-22
updated: 2026-08-22
tags:
  - agents
  - mcp
  - skills
  - onboarding
---

# Agent Skills and Surfaces

Canonical map of **what AI agents can use with Diamante**, and **where each kind of knowledge or capability belongs**. Locks the layering so host-agent “skills,” MCP tools, and vault notes are not mixed up.

## Related

- [[Agent and Human Setup]]
- [[Command Palette and Agent Surface]]
- [[Unified Knowledge System]]
- [[Next Steps]]
- [[Home]]
- [[Frontend]]

## Principle

Diamante does **not** ship an in-app agent skill pack or a second full agent chat by default.

| Agents get | Via |
| --- | --- |
| **Tools** (query graph, read/write vault notes) | **MCP** (`scripts/diamante-mcp.mjs`) |
| **Hard rules + workflow** | App-repo **`AGENTS.md`** |
| **Product intent / roadmap** | **Docs vault** notes (this folder) |
| **Host playbooks** (how Hermes builds Diamante) | **Host skills** (e.g. Hermes `diamante-notes`) — not the Diamante product surface |

Product layering: **external agent + Diamante MCP**. Chat gateways (Slack/Telegram) stay Hermes territory.

## Three layers

| Layer | What it is | Where it lives | Consumer |
| --- | --- | --- | --- |
| **1. Diamante agent surface** | Runtime tools + contract | App MCP + app `AGENTS.md` | Any MCP client (Hermes, Cursor, Codex, Claude Desktop, …) |
| **2. Host-agent skills** | How to *work on* Diamante (build, index, chrome, ship) | Hermes skill store (or Cursor/Claude skill dirs) | Coding agents on the developer machine |
| **3. Vault knowledge** | Intent, decisions, domain notes | Configured vault Markdown | Humans + agents via MCP note tools / graph |

Do not put Layer 2 playbooks into the app as the primary third-party agent API. Do not treat vault design essays as MCP tools.

## Layer 1 — Diamante agent surface

### Contract

| File | Role |
| --- | --- |
| **`C:\Sites\diamante\AGENTS.md`** (app repo) | **Canonical** agent contract: hard rules, full MCP tool table, workflow, paths, validation |
| **This vault’s `AGENTS.md`** | Thin docs-folder / Codex pointer only — **not** the full tool contract |
| **`AGENTS-GROK.md` (this vault)** | **Pattern reference** (root contract + optional skills tree). Its tech stack does **not** apply to Diamante |

Design decision ([[Command Palette and Agent Surface]]): keep **one** app `AGENTS.md`, not a Diamante-repo `skills/` tree of one-file-per-tool, while the tool surface stays modest. Revisit only if tool count grows a lot.

### Runtime (MCP)

| Piece | Location |
| --- | --- |
| Server | `C:\Sites\diamante\scripts\diamante-mcp.mjs` |
| Helpers | `scripts/graph-lib.mjs` |
| Project hint | app `.mcp.json` |
| Human export | Setup wizard / Settings → Agent setup / palette **Copy MCP config for agents** |

**Env:** one vault per process — `DIAMANTE_VAULT_DIR` (required for real use), optional `DIAMANTE_PROJECT_DIR`.

**Typical multi-vault setup (this machine):**

| MCP server name | Vault |
| --- | --- |
| `diamante-dakila` | `C:\Users\webwi\Documents\Dakila` |
| `diamante-docs` | `C:\Users\webwi\Documents\Diamante` |
| (optional) `diamante-qwen` | Qwen notes vault |

### Tool groups (same tools, vault-bound instance)

| Kind | Tools |
| --- | --- |
| Graph read | `query_graph`, `get_node`, `get_neighbors`, `shortest_path`, `god_nodes`, `search_symbols`, `explain_edge`, `impact_of`, `list_communities`, `graph_stats` |
| Graph durability | `rebuild_graph` (refresh `.diamante/graph.json`; clear stale) |
| Notes read | `list_notes`, `search_notes`, `read_note` + resources `diamante://note/…` |
| Notes write | `create_note`, `write_note`, `rename_note`, `delete_note` (soft → `.diamante/trash/`) |

### Hard rules (non-negotiable)

- Write **vault notes only** through MCP — never the indexed **source root**
- One `DIAMANTE_VAULT_DIR` per MCP process — no cross-vault ops in one server
- Path containment (`safeJoin`); escapes fail clearly
- Soft delete via MCP trash — not hard delete
- Prefer `.diamante/graph.json` when present; after writes check `graph_stats.stale` and call `rebuild_graph` when durable freshness matters
- Repo re-index signature is `branch|HEAD` only — dirty trees must not thrash the indexer

### Recommended agent workflow

1. Orient — `graph_stats` (and `stale` / age)
2. Find — `query_graph` / `search_notes` / `search_symbols` / hubs / communities
3. Read — `read_note` or note resource; for source use graph `sourcePath` + host tools (not MCP write)
4. Relate — `get_node`, `get_neighbors`, `shortest_path`, `explain_edge`, `impact_of`
5. Act (vault only) — create / write / rename / soft-delete
6. Refresh — `rebuild_graph` if other agents or the app need a durable artifact update

### Human app surface (not agent skills)

Command palette (`Cmd+K`) names existing **UI** actions for people. It is not a skill pack for external agents. Semantics should stay aligned with MCP where both exist (e.g. note ops).

## Layer 2 — Host skills (Hermes example)

Procedural playbooks for **developers and coding agents** working on Diamante. Not end-user product features.

| Skill (Hermes) | When |
| --- | --- |
| `diamante-notes` | App/code, graph indexer, engines, Tauri, UI chrome |
| `diamante-product-delivery` | Roadmap, docs vault planning, MCP onboarding, OTA/landing |
| `dakila-overland` | Separate vehicle product — not Diamante app |

**Where to put new Hermes skills for Diamante work (this machine):**

```text
%LOCALAPPDATA%\hermes\skills\software-development\<name>\SKILL.md
  references\...
```

Profile-only skills:

```text
%LOCALAPPDATA%\hermes\profiles\<profile>\skills\...
```

Versioning host skills in the Diamante git repo is optional and secondary; third-party agents should rely on **MCP + app `AGENTS.md`**.

## Layer 3 — Vault knowledge

| Content | Vault |
| --- | --- |
| Product roadmap, architecture, this map | **Docs vault** (`Documents\Diamante`) |
| User/project notes agents should edit | **Opened user vault** (e.g. Dakila) via MCP |
| Design essays tagged “skill” (e.g. [[Frontend]]) | Docs vault guidance — **not** MCP runtime |

Agents discover vault content through MCP note tools and the graph, not by scanning a host `skills/` folder.

## Placement decision tree

```text
Is it a tool (list notes, query graph, write note)?
  → MCP in app repo + row in app AGENTS.md

Is it a hard rule / workflow for any agent using Diamante?
  → App AGENTS.md (single contract)

Is it product planning / “why we built X”?
  → Docs vault note + wikilinks (Home / Next Steps / Agent and Human Setup)

Is it a repeatable host coding procedure (indexer, tauri build, chrome)?
  → Hermes diamante-notes or diamante-product-delivery (+ references/)

Is it domain knowledge for a user project?
  → That user’s vault notes (+ attach/index repo for graph)

Many small one-tool playbooks inside Diamante?
  → Don’t — keep one AGENTS.md until the tool surface is large
```

## Vault AGENTS.md for end-user projects

Diamante can **prepare a vault-scoped agent contract** so other users' agents (Hermes, Cursor, Codex, …) know MCP rules for **that** project — not the Diamante app-repo contract.

| Behavior | Detail |
| --- | --- |
| **When** | Setup wizard **Agents** step, Settings → Agent setup, palette **Add or refresh agent contract** |
| **Not** | Silent auto-write on every vault open |
| **Create** | If no root `AGENTS.md` → create vault-root `AGENTS.md` with managed markers |
| **Refresh** | If markers present → replace only between `<!-- diamante-agent-contract:start/end -->` |
| **Collision** | If root `AGENTS.md` exists without markers → write `.diamante/AGENTS.md` (never clobber) |
| **Merge (opt-in)** | Append managed block into existing root `AGENTS.md` |
| **Content** | Short vault MCP workflow + hard rules; user prose outside markers is preserved |

App implementation: `src/agentContract.ts`, `SetupWizard` Agents step, Settings buttons, command palette.

Resolution order for agents: managed root `AGENTS.md` → else `.diamante/AGENTS.md` → else missing (Setup CTA).

## Explicit non-goals

- In-app second full agent chat as the default agent surface
- Diamante-native skill marketplace or skill loader
- MCP mutation of indexed source roots
- Cross-vault ops in a single MCP process
- Treating `AGENTS-GROK.md` or host skill trees as Diamante’s runtime API
- Shipping Hermes profile skills as the end-user install story (P7 is OTA / no-Node MCP, not skills)

## Paths (this Windows machine)

| Role | Path |
| --- | --- |
| App / code | `C:\Sites\diamante` |
| Docs vault | `C:\Users\webwi\Documents\Diamante` |
| Example notes vault | `C:\Users\webwi\Documents\Dakila` |
| MCP script | `C:\Sites\diamante\scripts\diamante-mcp.mjs` |
| Graph artifact | `<opened-vault>\.diamante\graph.json` |
| App agent contract | `C:\Sites\diamante\AGENTS.md` |

Historical macOS / `Documents/Projects/Diamante` paths may appear in older notes — prefer the table above on this machine.

## Implementation pointers

| Piece | Location |
| --- | --- |
| MCP server | `scripts/diamante-mcp.mjs` |
| MCP tests | `scripts/diamante-mcp-write.test.mjs`, related |
| MCP export UI | `src/mcpExport.ts`, `src/SetupWizard.tsx` |
| Command palette | `src/CommandPalette.tsx`, `src/commands.ts` |
| Agent contract | app-repo `AGENTS.md` |

## Acceptance (map is “done”)

- [x] Layers 1–3 named and separated
- [x] Placement tree for new tools vs docs vs host skills
- [x] Linked from [[Home]], [[Agent and Human Setup]], [[Next Steps]], [[Command Palette and Agent Surface]]
- [x] One-line pointer in app `AGENTS.md` → this note title (docs vault)
