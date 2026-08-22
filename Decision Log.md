---
id: nodez-decision-log
title: Decision Log
type: decision-log
status: active
created: 2026-08-19
updated: 2026-08-22
tags:
  - decisions
---

# Decision Log

## 2026-08-22 — Brand Nodez + getnodez.app

- User-facing product name is **Nodez** (not Nodez).
- Public domain **getnodez.app** is the canonical site + OTA base.
- Vault meta directory is **`.nodez/`** (legacy **`.diamante/`** still readable). MCP env `NODEZ_*` retained for agent configs; agent-contract markers accept both nodez and legacy diamante forms.
- App identifier `app.getnodez.nodez`. GitHub repository renames deferred.

## 2026-08-22 — Signing env for tauri build

**Decision:** Document and use `TAURI_SIGNING_PRIVATE_KEY` = **private key file contents** for `npm run tauri -- build`. Do not rely on `TAURI_SIGNING_PRIVATE_KEY_PATH` for bundling (works for `tauri signer sign` only on this toolchain).

**Also:** Landing `updates/bundles/*` stays gitignored for local junk; release CI and intentional publishes use `git add -f` so Pages can host installers.

## 2026-08-22 — Landing split to public repo

**Decision:** Move the public download site + OTA feed out of the private app repo into **`paulbrett/nodez`** (public), so GitHub Pages works on the free plan.

**Why:** Private `nodez` cannot use Pages without Pro. User asked for a separate landing repo.

**Implications:** App updater endpoint → `https://paulbrett.github.io/nodez/updates/latest.json`. Release CI needs `LANDING_DEPLOY_TOKEN` to push feed/bundles. Local `NODEZ_LANDING_DIR` defaults to sibling `../nodez`.

## 2026-08-22 — Landing lives in main repo as plain HTML/CSS

**Decision:** Public download site is `landing/` inside `C:\Sites\nodez-app`, plain HTML + CSS (tiny JS only to soft-fill version from the updater manifest). OTA manifest and update bundles co-locate under `landing/updates/`. Host on GitHub Pages at `paulbrett.github.io/nodez`.

**Why:** User choice (single-page, in-repo, manifest + bundles together). Avoids a second repo and keeps release CI one place.

**Implications:** Pages workflow deploys `landing/`; release workflow copies signed updater artifacts into `updates/bundles/` and rewrites `latest.json`. App updater endpoint points at that static JSON.

## 2026-08-22 — Vault AGENTS.md collision policy

**Decision:** Opt-in create/refresh of vault agent contract; never full-overwrite existing root `AGENTS.md`. Managed HTML comment markers; foreign root → `.nodez/AGENTS.md`; optional merge-append.
**Why:** Root `AGENTS.md` is a common host-agent convention; clobbering user/team contracts is unacceptable.
**See:** [[Agent Skills and Surfaces]]

## 2026-08-22 — Agent skills vs MCP surfaces

**Decision:** Nodez does not ship an in-app skill loader. Agents use **MCP + app `AGENTS.md`**. Host skills (Hermes) and vault notes are separate layers. Documented in [[Agent Skills and Surfaces]].
**Why:** Matches product layering (external agent + MCP); avoids a second skill tree competing with the tool contract.

## 2026-08-19 - Use plain Markdown files

Nodez should keep notes as normal `.md` files.

Reason: portability is the central promise.

## 2026-08-19 - Use GitHub instead of proprietary sync

Sync should be implemented through git and GitHub.

Reason: it gives version history, user ownership, and avoids building a cloud sync service.

## 2026-08-19 - Start with React and CodeMirror

The first prototype uses React, TypeScript, Vite, and CodeMirror.

Reason: this is a fast way to build the editor and workspace UI.

## 2026-08-19 - Prefer Tauri for desktop

Tauri is the current preferred shell for the next phase.

Reason: it fits the local-first file access and native git direction.

## 2026-08-19 - Combine Obsidian and Graphify Models

Nodez should combine Obsidian's local Markdown vault workflow with Graphify's project relationship graph workflow.

Reason: Dakila needs a unified system where human-readable decisions, implementation truth, and machine-queryable relationships stay aligned.

## 2026-08-20 - Use vis-network for the Graphify-compatible graph UI

The full graph modal should use `vis-network@9.1.6`, matching Graphify's
open-source HTML exporter, instead of a separate custom force-graph renderer.

Reason: Nodez should inherit Graphify's proven graph interaction semantics:
ForceAtlas2-style physics, degree-sized dot nodes, community coloring, arrows,
search/focus behavior, click inspection, and confidence-styled edges.

## 2026-08-20 - Replace vis-network with an in-house canvas renderer

The graph modal should render with a dependency-free canvas force engine
(`src/GraphifyNetwork.tsx`) instead of `vis-network`, kept as a true drop-in for
the same props.

Reason: it fits Nodez's local-first, minimal-dependency stance (no external
graph library, works offline), and a Barnes-Hut quadtree keeps the 1,000-node
demo smooth. This supersedes the earlier vis-network decision; `vis-network` is
now unused and can be removed from `package.json`.

## 2026-08-20 - Use the Dakila Overland Controller repo as the first external repo-indexing target

When Nodez starts indexing an external project/repo folder (not just its own notes vault), the first concrete target is `/Users/paulbrettorozco/Sites/overland/OverlandLightingControllerV1`, in favor of the sibling `dakila-landing` static site in the same `/Sites/overland` folder.

Reason: it is the substantive engineering repo — ESP32 firmware (C++), a Node.js backend, and a React Native/TypeScript mobile app, plus its own `AGENTS.md` and `docs/` folder — a real multi-language, multi-component codebase, and the same repo the Dakila-style workflow in [[Unified Knowledge System]] was written for. See [[Repo Indexing]] for the phased plan.

## 2026-08-20 - Tauri commands that call a blocking dialog API must be `async fn`

Any Tauri command that calls a `tauri-plugin-dialog` blocking method (`blocking_pick_folder`, `blocking_pick_file`, etc.) must itself be declared `async fn`.

Reason: a plain (non-`async`) command's body runs synchronously on the same thread that handles the IPC call — the main thread on macOS. The blocking dialog methods are explicitly documented as unsafe to call from the main thread, since showing/dismissing the native panel needs that same thread's run loop free. Calling one from a non-async command self-deadlocks the whole app (this is exactly what happened with `pick_vault`, fixed the same day). Any future folder/file picker command (e.g. picking a source root for [[Repo Indexing]]) must follow this rule from the start.

## 2026-08-20 - Use Tauri v2 for the desktop shell

The desktop shell is Tauri v2. Vault filesystem access is implemented as Rust
commands, with a frontend adapter (`src/vault.ts`) that falls back to
localStorage in the browser.

Reason: it confirms the earlier "prefer Tauri" direction; v2 is current, ships a
small bundle, gives native filesystem access and file watching in Rust, and
keeps the editor unaware of whether notes come from localStorage or disk.

## 2026-08-20 - Store the unified graph artifact inside the opened vault

Nodez should write the merged vault+repo graph to `.nodez/graph.json`
inside the opened vault by default.

Reason: the graph is portable with the vault, easy for AI agents to discover,
and still clearly a generated index rather than source truth. The vault notes
remain the human-readable intent layer; the repo remains implementation truth;
`.nodez/graph.json` is the machine-readable relationship map that ties them
together.

## 2026-08-20 - Use git status polling as the first source-root watcher

Phase 6a watches an indexed source root by polling `git -C <root> status` and
HEAD every 7 seconds, then re-indexing when the signature changes.

Reason: it gives immediate practical change detection for repos without adding
a second full filesystem watcher yet. It also aligns with the product direction
that source roots are git-backed implementation truth. A native recursive source
watcher can still be added later for non-git folders or finer-grained updates.

## 2026-08-20 - Prefer a calm workspace shell over a dashboard shell

Nodez's main workspace should follow the Obsidian pattern: notes and editor
stay central, secondary workspace actions live as compact icon buttons, and
ambient system state lives in a quiet status bar.

Reason: the product is a daily thinking and editing surface, not a reporting
dashboard. Graphify-style density belongs inside the graph modal where users
are actively inspecting relationships; the everyday workspace should stay
focused on the vault, the active note, and nearby backlinks/context.

## 2026-08-21 - Re-index attached repos only on commits, not dirty worktrees

`GitRepoState.signature` is `branch|HEAD` only. Porcelain dirty/untracked status
may still populate the status-bar summary, but it must not change the signature
or restart repo indexing. Poll interval is about 15 seconds.

Reason: indexing on every uncommitted save made the graph feel permanently
"loading" and burned CPU on large repos. Implementation truth for the graph is
committed code; local scratch should not thrash the indexer. Manual re-index
from the status bar remains available.

Supersedes the earlier interpretation of the 2026-08-20 git poll decision that
bundled porcelain into the re-index signature.

## 2026-08-21 - Persist editor mode and graph toggles per vault

Edit/Preview plus graph mode, engine, origin, depth, node/edge type, and provenance
filters persist in vault meta (`.nodez`) and browser `localStorage`
(`nodez.uiPrefs`). `indexRepo` must not reset graph origin/mode to defaults.

Reason: users switch edit/preview and graph filters constantly; losing them on
reload or re-index breaks the calm workspace promise.

## 2026-08-21 - Graph modal title is the repo name

The main graph modal title shows the attached source-root folder name (else the
vault folder name, else "Connections").

Reason: when a project repo is open, the graph is about that codebase — the
title should say so instead of a generic "Knowledge connections" label.

## 2026-08-21 - Dual graph engines: Canvas 2D default + optional 3D force

The graph modal exposes a user-selectable **graph engine** toggle:

- `canvas2d` — in-house `GraphifyNetwork.tsx` (default; no extra runtime cost)
- `force3d` — `react-force-graph-3d` / Three.js via `ForceGraph3DNetwork.tsx`

Both engines share the same props contract (selection, path highlight, filters
unchanged). Preference persists as `graphEngine` in vault meta and
`nodez.uiPrefs`. The 3D package is **lazy-loaded** so Canvas 2D users do not
pay the WebGL bundle until they switch.

Reason: 3D force layouts help explore dense relationship graphs, but the
default path should stay lightweight and offline-friendly. This extends (does
not replace) the 2026-08-20 in-house canvas decision.

## 2026-08-21 - Graph toolbar uses flex layout

`.graphControls` is flex with wrapping, not a fixed column grid, so additional
icon groups (mode / engine / origin) do not break search and filters.

Reason: the previous 4-column grid broke when the engine toggle was added.

## 2026-08-21 - Prioritize agent completeness + human setup before more UI chrome

After palette + MCP writes + dual Hermes servers, the next product track is
[[Agent and Human Setup]] (P0–P3 before P4–P7):

1. MCP note read tools and resources
2. Graph artifact freshness after agent writes
3. First-run wizard and one-click MCP config export
4. App-repo `AGENTS.md` as a real agent contract

Then impact/communities tools, deeper Dakila extraction, real git sync, and
no-Node MCP distribution.

Reason: agents cannot close the loop without reading notes and trusting the
graph; humans will not adopt MCP if every machine needs hand-edited paths.
In-app second agent chat and chat gateways stay out of scope — external agent +
Nodez MCP is the layering decision.

## 2026-08-21 - Ship versioning, OTA, and a landing page as distribution work

Public distribution is not only "run tauri build". Nodez should have:

1. Single app version source of truth and visible About version
2. Signed OTA updates via Tauri updater (user consent; air-gap opt-out)
3. A simple public landing page for download and positioning

Documented in [[Distribution Versioning and Updates]] and [[Landing Page]]. Sequencing stays under Phase 5 / P7 and must not block agent P0–P3; landing and release feed should land together when first public downloads are offered.

Reason: without versioning and updates, every human reinstall is friction; without a landing page, installers and OTA feeds have no front door.
