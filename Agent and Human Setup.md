---
id: diamante-agent-human-setup
title: Agent and Human Setup
type: roadmap
status: active
created: 2026-08-21
updated: 2026-08-22
tags:
  - agents
  - mcp
  - onboarding
  - roadmap
---

# Agent and Human Setup

Plan for making Diamante **useful for AI agents** and **simple to set up for humans**. Canonical follow-on after command palette + MCP write tools and dual Hermes servers (`diamante-dakila`, `diamante-docs`).

## Related

- [[Command Palette and Agent Surface]]
- [[Agent Skills and Surfaces]]
- [[Unified Knowledge System]]
- [[Next Steps]]
- [[Product Roadmap]]
- [[Backlog]]
- [[Repo Indexing]]
- [[Distribution Versioning and Updates]]
- [[Landing Page]]

## Current baseline (2026-08-21)

### Humans

- Real vault on disk (Tauri), note tree, editor/preview, graph (Canvas 2D default + optional 3D)
- Attach read-only source root; commit-driven re-index; `.diamante/graph.json` (+ chunks)
- Command palette (`Cmd+K`); MSI/NSIS installers exist for Windows

### Agents

- Stdio MCP (`scripts/diamante-mcp.mjs`): graph query tools + vault note create/write/rename/delete (soft trash)
- Prefers saved graph artifact over Markdown-only rebuild
- Hermes can run one server instance per vault (env: `DIAMANTE_VAULT_DIR`, optional `DIAMANTE_PROJECT_DIR`)

### Gaps that block “done enough”

1. ~~MCP can write notes but cannot **list / search / read** them as first-class tools~~ — **done (P0):** `list_notes` / `search_notes` / `read_note` + `diamante://note/...` resources
2. ~~Agent writes do not keep `.diamante/graph.json` as fresh as the app indexer~~ — **done (P1):** `graph_stats.stale` + `rebuild_graph`; in-memory note-layer merge after writes so queries see new notes before rebuild
3. ~~Human MCP wiring still needs hand-edited absolute paths and Node on PATH~~ — **done (P2):** setup wizard + one-click MCP JSON export with absolute vault/script paths; Windows export now strips `//?/` / `\\?\` extended prefixes and can use `DIAMANTE_NODE_COMMAND` when `node` is not on the client PATH
4. ~~App-repo `AGENTS.md` is not yet a full agent contract (tool table + hard rules + workflow)~~ — **done (P3):** app-repo `AGENTS.md` is the agent contract
5. ~~Higher-order graph tools (`explain_edge`, `impact_of`, communities) and deep code edges still thin~~ — **P4 tools done** (`explain_edge` / `impact_of` / `list_communities`); deep code edges still P5

## Skills vs surfaces

Agents do **not** load a Diamante in-app skill pack. Runtime capability is **MCP tools** + app-repo `AGENTS.md`. Host playbooks (Hermes `diamante-notes`, etc.) and vault notes are separate layers. Canonical map: [[Agent Skills and Surfaces]].

### Vault agent contract (end-user projects) — shipped 2026-08-22

Opt-in **AGENTS.md** for the opened vault so external agents get project MCP rules:

- Setup wizard step **Agents** (between Index and MCP)
- Settings → Agent setup: **Agent contract** / **Merge into AGENTS.md**
- Palette: **Add or refresh agent contract**, **Merge Diamante section into existing AGENTS.md**
- Collision: never overwrite foreign root `AGENTS.md`; fallback `.diamante/AGENTS.md`; managed marker refresh

See [[Agent Skills and Surfaces]].

## Definition of done

**Humans:** install → open vault → attach repo → index → copy MCP config → agent works without editing env by hand.

**Agents:** query graph → read the note/file the graph cites → write a vault note → see updated relationships (or a clear rebuild signal) — without mutating the source root through MCP.

## Priority sequence

Ordered by leverage. Do not reorder casually; later items assume earlier ones.

### P0 — Agent read surface — **shipped 2026-08-21**

MCP note reads land in `scripts/diamante-mcp.mjs` (+ `scripts/diamante-mcp-write.test.mjs`).

| Tool / resource | Behavior |
| --- | --- |
| `list_notes` | Vault-relative paths + titles + mtime + note URI |
| `search_notes` | Title/path/content snippet search, bounded results (default 20, max 100) |
| `read_note(path)` | Full Markdown body; path-containment via `safeJoin` |
| Resources | `diamante://note/<encoded-path>` listed/readable alongside graph stats/hubs |

Hard rules unchanged: vault notes only; never write the indexed source root through this surface.

### P1 — Graph freshness after agent edits — **shipped 2026-08-21**

Writes reset the MCP in-process cache **and** set `vaultDirty`. Queries merge a live vault note layer over the durable artifact so new notes appear immediately. Durable freshness:

1. **`graph_stats`** includes `stale`, `reasons`, `artifactAgeMs`, `artifactMtimeMs`, `vaultNoteMtimeMs`, `sourceHead`, and a `rebuild` hint
2. **`rebuild_graph`** refreshes the vault note layer into `.diamante/graph.json` (Graphify-shaped export) and clears `stale`

Document the model in app-repo `AGENTS.md` under P3 and [[Command Palette and Agent Surface]].

### P2 — First-run wizard + one-click MCP export (human setup) — **shipped 2026-08-21**

In-app guided flow (`SetupWizard` + Settings → Agent setup):

1. Open or create vault
2. Attach repo (optional)
3. “Index for AI” (metadata; optional symbols when graph opens — existing behavior)
4. **Export MCP config** for Hermes / Claude Desktop / Cursor (copy JSON; pre-filled absolute vault + script path when resolvable)
5. Smoke line: “Graph ready · N nodes · MCP command = …”

Rust `resolve_mcp_paths` finds `scripts/diamante-mcp.mjs` (dev manifest path, exe-adjacent, or `DIAMANTE_MCP_SCRIPT`). Command palette: **Open setup wizard**, **Copy MCP config for agents**.

Codex Desktop lesson applied 2026-08-21:

- Codex MCP config lives in `%USERPROFILE%\.codex\config.toml` as `[mcp_servers.<name>]`, not JSON `mcpServers`
- Plain Windows paths (`C:/Sites/diamante/scripts/diamante-mcp.mjs`) work; extended paths (`//?/C:/...` or `\\?\C:\...`) failed under Node
- Desktop clients may not inherit a shell PATH, so `node` can fail even when a bundled/runtime Node exists; use `DIAMANTE_NODE_COMMAND` or an absolute `node.exe`
- Restart Codex Desktop or start a new task after changing MCP config

Optional later: “Open Hermes Capabilities” deep link if the host supports it.

### P3 — Agent contract docs — **shipped 2026-08-21**

| Surface | Work |
| --- | --- |
| App repo `AGENTS.md` | **Done** — what this is; hard rules table; full MCP tool reference; workflow (graph → notes → source → act); paths; validation |
| This vault | Keep product intent here; link from [[Home]] / [[Next Steps]] |
| Paths | Prefer Windows-real paths on this machine (`C:\Sites\diamante`, `Documents\Diamante`) and note macOS dual-path history only where needed |

### P4 — Higher-order graph tools + resources — **shipped 2026-08-21**

Align MCP with research/roadmap names:

- `explain_edge` — edge type/provenance/confidence/sourcePath between two nodes (or by `edge_id`)
- `impact_of` — downstream fan-out within N hops
- `list_communities` — communityName buckets with sizes/samples
- Resources: hubs, stats, **communities**, note URIs

Deeper extraction remains [[#P5 — Deeper extraction (Dakila proving ground)]].

### P5 — Deeper extraction (Dakila proving ground) — **regex pass landed 2026-08-22**

- Cross-file calls, multi-line imports, methods — regex extractor; Dakila smoke OK — [[Repo Indexing]]
- Optional later: tree-sitter WASM for full syntax fidelity
- Keep commit-only re-index signature (`branch|HEAD`); do not thrash on dirty trees

### P6 — Real GitHub sync + rebuild-after-pull — **v1 landed 2026-08-22**

Rust vault git + sync panel (notes footer control):

- status, pull (`--ff-only` then merge), stage, commit, push, sync
- stops on conflicts (no silent overwrite); notes reload after pull/sync
- Still later: guided conflict editor, clone/connect wizard, OAuth, auto graph rebuild on pull

### P7 — Distribution polish — **live OTA path 2026-08-22**

**Landed:**

- Installers: MSI/NSIS via Tauri; version `0.3.0` in package/tauri/Cargo
- **Landing** — separate **public** repo `paulbrett/diamante-landing` (`C:\Sites\diamante-landing`); Pages live — [[Landing Page]]
- **OTA** — signed feed at `https://getnodez.app/updates/latest.json` with `platforms.windows-x86_64` + bundles; Settings → About → Check for updates — [[Distribution Versioning and Updates]]
- Secrets: `TAURI_SIGNING_PRIVATE_KEY`, `LANDING_DEPLOY_TOKEN` on private app repo; local key `src-tauri/diamante.key`
- CI: landing Pages workflow; app `release.yml` force-pushes feed/bundles; `scripts/publish-update-feed.mjs`

**Still remaining:**

- Tag GitHub Release + verify CI end-to-end; N−1 → N OTA smoke test
- [x] App icons regenerated from 1024 brand PNG
- Authenticode / macOS notarization when those platforms ship
- **No-Node MCP path**: bundle portable MCP or expose from Tauri binary
- Keyboard shortcuts beyond palette; fast note search; attachments (Phase 3)

#### Related distribution notes

- [[Distribution Versioning and Updates]]
- [[Landing Page]]

## Explicit non-goals (for this track)

- In-app second full agent chat by default — prefer external agent + Diamante MCP ([[Command Palette and Agent Surface]] product layering)
- Slack/Telegram as Diamante features — Hermes gateway territory
- MCP write access to indexed source roots
- Cross-vault MCP in one process (one `DIAMANTE_VAULT_DIR` per instance)
- Perfect Graphify parity before P0–P3

## Acceptance checks

### P0

- [x] Agent can list/search/read a note without shelling into the vault folder
- [x] Path escape attempts fail with a clear error
- [x] `npm` MCP write tests extended or mirrored for read tools

### P1

- [x] After `write_note`, either artifact updates or `graph_stats` reports stale with how to rebuild
- [x] Hermes smoke: write → query mentions new content or stale flag

### P2

- [x] New user (or clean profile) can copy a working MCP JSON from the app without hand-editing paths
- [x] Snippet includes the opened vault path and resolved Node command + `diamante-mcp.mjs` (or a clear script-path placeholder)
- [x] Windows export does not emit Node-breaking `//?/` or `\\?\` script paths

### P3

- [x] App `AGENTS.md` documents all MCP tools and hard rules
- [x] Docs vault links this note from [[Next Steps]] and [[Home]]

### Done-enough (product)

- [ ] Human path: install → vault → repo → index → copy MCP → agent answers a graph question and updates a note
- [ ] Agent path never needs write access to the source root

## Implementation pointers (app repo)

| Piece | Location |
| --- | --- |
| MCP server | `scripts/diamante-mcp.mjs` |
| MCP write tests | `scripts/diamante-mcp-write.test.mjs` |
| Project MCP hint | `.mcp.json` |
| Command palette | `src/CommandPalette.tsx`, `src/commands.ts` |
| Graph artifact | vault `.diamante/graph.json` + `.diamante/graph/` |
| Workspace bind | vault `.diamante/workspace.json` |

App/code home on this machine: `C:\Sites\diamante`. Docs vault: `C:\Users\webwi\Documents\Diamante`.
