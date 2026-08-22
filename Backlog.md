---
id: diamante-backlog
title: Backlog
type: backlog
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - backlog
---

# Backlog

Priority track for agents + onboarding: [[Agent and Human Setup]]. Sequencing: [[Next Steps]].

## Editor

- Add slash command menu
- Add formatting toolbar — done (Markdown tools toggle)
- **Plain-text / paste auto-format** — detect unformatted paste or body; ask format-or-not; show Auto-format button when unformatted — [[Next Steps]]
- **Lightweight code editor** (CodeMirror 6 + Prettier format + light lint; not Monaco for MVP) — [[Code Editor Implementation]], [[Next Steps]]; builds on `src/MarkdownEditor.tsx`
- Add image and file embeds
- Add heading outline — done (inspector outline)
- Add keyboard shortcuts (beyond command palette)

## Vault

- Open local folder — done
- Create note in folder — done
- Rename note and update links — done
- Delete note with confirmation — done
- Move notes between folders — done
- Create empty folders — done
- First vault load collapses folders by default — done

## Sync

- Show git status — done (vault git v1)
- Pull from remote — done
- Commit local changes — done
- Push to GitHub — done (v1)
- Conflict resolution screen — stop + list done; guided editor later
- Rebuild `.diamante/graph.json` after successful pull — notes reload; durable auto-rebuild later

## Index / Graph

- Better tag parsing
- Unresolved links
- Backlink snippets
- Fast search ranking
- Graph filters and zoom controls — largely done
- Graph search and connection highlighting — done
- Expand node and edge schema beyond notes/tags — done
- Add inferred and manual edge provenance — done
- Local graph depth control — done
- Query, path, and explain graph tools — done in UI
- Source references from graph edges back to notes/files/lines — partial
- Lazy-load heavy graph engines — done for WebGL 3D
- Add graph performance measurements using the 1,000-node dummy vault fixture
- Layout cache (`.diamante/layout.json`) and path/impact worker — **done** — [[Graph Scale]]
- Optional community hulls; minimap / zoom-to-fit — zoom-to-fit landed; hulls/minimap optional

## Agent / MCP (see [[Agent and Human Setup]])

- Graph query MCP tools — done
- Vault write tools (create/write/rename/soft-delete) — done
- Dual Hermes vault servers — done
- `list_notes` / `search_notes` / `read_note` — **P0**
- Note MCP resources (`diamante://note/...`) — **P0**
- Graph freshness after writes / `rebuild_graph` + stale signal — **P1**
- First-run wizard + one-click MCP JSON export — **P2**
- Restructure app-repo `AGENTS.md` agent contract — **P3**
- `explain_edge`, `impact_of`, `list_communities` — **P4**
- Workspace binding resource (attached repo, HEAD, last indexed) — **P4**
- No-Node / bundled MCP binary — **P7**

## Distribution (see [[Distribution Versioning and Updates]], [[Landing Page]])

- Single version source of truth across npm + Tauri + About UI — done baseline (0.3.0)
- Semver tags and release checklist — tags + CHANGELOG; CI release workflow
- Windows MSI/NSIS — done
- Public landing page (`paulbrett/diamante-landing` Pages) — **live**
- OTA: signed `updates/latest.json` + Settings check — **live feed; tag CI verify remaining**
- First signed Pages feed (`platforms.windows-x86_64`) — pending secret + tag
- Authenticode / SmartScreen — pending
- No-Node / bundled MCP binary — **P7 remaining**
- Full icon set / macOS notarization — pending

## Dakila Workflow

- Open docs vault and source repo as one workspace — done (vault + attached source root)
- Graph query before broad file search — available via MCP; agent contract still P3
- Source-of-truth conflict banner
- Post-change reminder to update docs and graph
- GitHub sync flow for code/docs/graph outputs

## Repo Indexing (see [[Repo Indexing]])

- Read-only source-root command — done
- Register a source root alongside the vault path — done
- `file`/`folder` graph nodes + containment — done
- Save merged graph artifact at `.diamante/graph.json` — done
- Visible Open Vault/Open Repo controls — done
- Markdown docs + `package.json` into documents/references/depends_on — done
- Preserve extracted repo metadata in artifact — done
- Commit-only re-index signature — done
- Tree-sitter (or equivalent) extraction for C++ and TypeScript/TSX — **P5**
- MCP read surface covers vault + source graph (writes vault-only) — extend with P0 reads
- Filesystem watcher + rebuild for non-git source roots (optional later)

## Design / Shell

- Expand and refine color themes — done baseline set
- Empty states — partial
- Settings screen — done
- Command palette — done
- Light and dark themes — done
- First-run / empty-vault onboarding wizard — **P2**
- About / version display in Settings — [[Distribution Versioning and Updates]]
- Check for updates UI — [[Distribution Versioning and Updates]]
