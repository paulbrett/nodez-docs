---
id: nodez-decision-log
title: Decision Log
type: decision-log
status: active
created: 2026-08-19
updated: 2026-09-07
tags:
  - decisions
---

# Decision Log

## 2026-09-07 — Plan code editor with AI chat

User requested a plan for an integrated code editor with AI chat. [[Code Editor and AI Chat Plan]] records the verified baseline, proposed phases, acceptance criteria, and durable project context. This is planning only: optional conversational editor assistance is proposed; a full autonomous agent and MCP source writes are not included. Repository editing requires a separate explicit native editor capability; indexed source attachment remains read-only. No application code changed.

## 2026-08-25 — Publishing is opt-in per page, enforced by a build that refuses

**Decision:** The public wiki publishes a manifest, not a folder. Adding a note to this vault does not publish it. The generator scans every rendered page for local identity — home paths, Windows user paths, signing-key references, tokens, third-party project names — and **fails the build** rather than emitting a warning.

**Why:** this vault is written for us, and the failure mode is one-directional. A page missing from the wiki is an inconvenience; a session log or a client's repository layout on a public marketing domain cannot be recalled. A guard that fails the build is the only kind that survives someone in a hurry.

**Corollary — judge a doc by its declared type, not its title.** `Agent and Human Setup` sounds like a setup guide and is a P0–P7 phase plan carrying CI secret names. Titles describe intent; frontmatter `type` describes what was actually written. Only `architecture`, `overview`, `sync`, `vault` and `workflow` may publish.

**Corollary — nonsense output means the content is machine-specific.** When scrubbing `Agent Skills and Surfaces` produced `%USERPROFILE%\Documents\the reference project`, that was not a scrub bug to fix. Content that cannot survive having local paths removed *is* local content. Drop the page; do not improve the regex.

## 2026-08-25 — Product documentation is written, not harvested

**Decision:** Public wiki pages live in `wiki-content/` in the app repo, authored for a reader. The vault-harvesting path stays in the generator but is currently unused.

**Why:** every doc here reads as an engineering spec once you look closely — a shortlist of search libraries, a table of Rust command names, `Non-goals`/`Data flow`/`Error handling` headings, and one sentence that trailed off into an empty code block. That is correct for a planning vault and wrong for someone who has just found the project. Harvesting also inherits contradictions: `GitHub Sync.md` had two sections disagreeing about where the sync control lives.

**Cost, accepted:** the wiki is now a second thing to maintain, and `whats-new.md` does not track `CHANGELOG.md`. Generating it from there would reintroduce exactly the register the wiki exists to avoid, so it is updated by hand at release time.

## 2026-08-25 — Verify a landing deploy against the live URL, never the Actions tab

**Decision:** Confirm landing changes by requesting `https://getnodez.app/...` and checking the response, not by reading workflow status.

**Why:** `pages.yml` has failed on every push since 2026-08-22 — `configure-pages` returns `Not Found` because Pages is not enabled on the repo — while **Cloudflare** serves the site and deploys fine. A red run after every push trains everyone to ignore red runs. Either delete the workflow or enable Pages; until then the live URL is the only honest signal.

## 2026-08-24 — A UI state that suppresses information must be explicitly requested and explicitly escapable

**Decision:** Any mode that hides or dims data — path highlighting today, filters and focus modes tomorrow — may only be entered by a deliberate user action, and must offer a visible way back. Never prefill the input that arms such a mode.

**Why:** the 3D graph's "no colour" bug was exactly this rule being broken twice. The Path panel auto-filled a target, which armed path mode, which dimmed every off-path node — so the app silently entered an information-suppressing state on open. And once armed there was no control that cleared it; the only exit was picking a different node from a `<select>` that had no empty option. A prefilled default is not consent, and a mode with no exit reads as a bug even when the rendering is perfect.

**Corollaries:**

- A `<select>` whose empty state is meaningful needs an explicit empty `<option>`. With `value=""` and no match, the browser displays the *first* option as though it were chosen — the state looks set when it is not, which is what tempted the auto-fill in the first place.
- Clicking empty canvas is the escape gesture for canvas modes; it already cleared selection, so it now clears the path too.

## 2026-08-24 — A rule that governs whether the whole view stays legible gets its own named, tested seam

**Decision:** `isPathRequested` lives in `src/graphPathRequest.ts` rather than inline in `computeGraphView`.

**Why:** it was a three-clause boolean in the middle of a pipeline, and it decided whether the entire graph kept its colours. `scripts/force-graph-3d-color.test.mjs` covered the paint function thoroughly and passed throughout — the bug lived in the untested condition upstream. A leaf module is also the only shape `node --test` can import here: `graphView.ts` uses extensionless relative imports that Node's type-stripping will not resolve. Guarded by `scripts/graph-path-activation.test.mjs`.

**Generalisation:** when a test passes while the feature it covers is visibly broken, the defect is upstream of the seam under test. Look for the unnamed condition, not a deeper bug in the tested function.

## 2026-08-24 — One large heading per panel; section names are chrome

**Decision:** In the graph inspector the selected node or edge is the only 15px heading. Path, Connections, Hubs and Communities render as 11px uppercase micro-headers.

**Why:** four section names at title weight competed with the one thing the panel is about, so the column read as five equal blocks. Demoting them both clarifies and compacts. Applies to any inspector-style panel: the subject is the heading, the tools are labels.

## 2026-08-24 — Truncate paths toward whichever end carries the identity

**Decision:** Two shorteners, chosen by what the reader needs to tell rows apart. `shortenPathForDisplay` (both ends) for the recents menu, where the head distinguishes two similarly-named vaults. `shortenPathTail` (left-truncating, opening on a segment boundary) for the graph inspector, where every row shares the head and the filename identifies the row.

**Why:** one shortener cannot serve both. In the inspector, `shortenPathForDisplay` spent twelve of thirty-four characters on `/Users/paulb…`, which is identical on every row, while CSS's default right-clip hid the filename entirely. Neither renders the useful part. The full path always stays in the `title` attribute. Note the earlier finding still stands: never do this with CSS `direction: rtl`, which drags the leading separator to the visual end.

## 2026-08-24 — Derive accent hues from `--ink`, never hardcode them

**Decision:** UI that needs a hue the palette does not provide (direction coding, semantic status) computes it as `color-mix(in srgb, <fixed hue> N%, var(--ink))` rather than a fixed hex.

**Why:** the chrome palette is deliberately monochrome — `--teal`, `--gold`, `--blue` and `--accent` all alias `--primary` variants — and of the seven themes, `light` and `paper` are light. A fixed bright hue washes out on white. Mixing with `--ink` darkens it on light themes and lightens it on dark ones with no per-theme rules. Verified: teal resolves to `srgb(0.40 0.72 0.70)` on dark and `srgb(0.07 0.39 0.37)` on light.

**Exception:** the graph *canvas* keeps its own fixed palette, independent of chrome theme. That stays as-is.

## 2026-08-24 — Connection rows describe the neighbour, not the edge

**Decision:** The graph inspector's Connections list keys on the other endpoint — neighbour label first, relation demoted to a pill, duplicates collapsed into a count, click navigates to that neighbour.

**Why:** keying on the edge made every row identical. For a note's own outgoing links the relation is constant and `sourcePath` is the note already selected, so N edges rendered as N copies of the same row while omitting the one useful fact. The panel also sits above Hubs/Communities now: it describes the current selection, so it should not be below two workspace-wide summaries.

## 2026-08-24 — Vault git: notice, never unattended sync

**Decision:** Nodez surfaces "N behind" and offers a manual sync. It does **not** auto-commit, auto-pull or auto-push on a timer.

**Why:** an auto-push publishes on a schedule and can create merges the user never asked for. A background *fetch* is different — it only updates refs under `.git`, never the working tree — and is required for the notice to be truthful at all, so that one runs once per opened vault.

## 2026-08-24 — Node.js is a prerequisite; Nodez ships no runtime

**Decision:** Nodez does not bundle a Node runtime. Users install Node.js 20+ themselves. The app detects it and guides installation when missing.

**Why:**

- It never worked anyway. The shallow glob `resources/mcp/runtime/*` shipped only the runtime's top-level docs, never `bin/node`, so no installer ever carried Node.
- Fixing the glob to `runtime/**/*` breaks the build: the prepared `bin/` contains `corepack`/`npm`/`npx` as symlinks into an already-deleted temp directory.
- A bundled Node is single-arch, so it cannot serve a universal2 macOS app — Rosetta cannot translate arm64 to x86_64, meaning an Apple-Silicon-built runtime simply cannot run on Intel.
- It cost ~112MB for a dependency most target users (people wiring up AI agents) already have.

**Consequence:** the launcher must find Node robustly, because a client launched from Finder or the Start menu inherits a reduced PATH. Resolution order is `NODEZ_NODE_COMMAND` → PATH → common install locations, then actionable install guidance.

## 2026-08-24 — `src-tauri/resources/mcp/` is generated output

**Decision:** Treat that directory as build output. The source of truth for the launchers is the inline templates in `scripts/prepare-mcp-bundle.mjs`.

**Why:** `beforeBuildCommand` runs the generator on every build, which overwrites the launchers. Editing the files directly looks like it works and is silently reverted at the next build. The files are also tracked in git because `tauri dev` copies them as-is without running the generator — which is why their committed file mode matters for dev, and only for dev.

## 2026-08-24 — Never seed a 3D layout from the 2D layout cache

**Decision:** The 3D graph adopts the shared node layout cache only when that cache actually carries depth, judged over the whole cache rather than per node.

**Why:** The cache is written by both engines and the 2D engine stores no `z`. Seeding a missing `z` as 0 put every node on one plane, and a planar configuration is a fixed point of the force simulation — z-forces cancel by symmetry, so it never becomes 3D. Nodes lacking `z` are now left undefined so d3-force-3d seeds them on its own sphere.

**Judge the cache, not the node:** one node legitimately sitting at z=0 must not throw away an otherwise good 3D layout.

## 2026-08-24 — Do not reheat the simulation when registering a force

**Decision:** Register custom d3 forces on the 3D graph without calling `d3ReheatSimulation()`.

**Why:** Reheating desyncs three-forcegraph's own tick bookkeeping against `cooldownTicks`. After it, simulation positions stop being applied to the meshes and **every node renders at the origin** — a blank canvas with no error. Registering the force is sufficient; it lands before the first layout settles.

## 2026-08-24 — Radial shaping is 3D only

**Decision:** The spherical envelope force applies to the 3D canvas. The 2D canvas keeps its existing Barnes-Hut + spring + linear-gravity layout untouched.

**Why:** A matching circular force was implemented for 2D and reverted — the 2D layout was already considered good. Keeping it out avoids retuning a layout nobody complained about.

## 2026-08-23 — 3D graph auto-rotate orbits the camera, not the controls

**Decision:** Implement graph auto-rotation by rotating the camera around its own look-at target each frame, rather than switching to `OrbitControls` and using its `autoRotate`.

**Why:** `controlType` is an **init-only** prop in react-force-graph / three-render-objects, so `OrbitControls.autoRotate` cannot be a runtime toggle without remounting the whole graph. Orbiting the live camera also preserves whatever zoom, elevation and pan the user already has, which re-issuing `cameraPosition()` from a fixed radius does not.

**Consequence:** Scripted camera moves (`zoomToFit`, node focus) must claim the camera so the orbit loop yields — they mark a busy window, and the orbit also pauses while the pointer is down.

## 2026-08-23 — Bloom implies a darkened graph backdrop

**Decision:** While the bloom toggle is on, the 3D graph background drops from `#0a0e16` to near-black `#02040a`.

**Why:** `UnrealBloomPass` lifts the entire frame — its lowest mips spread bright pixels canvas-wide — so a normal backdrop reads as a grey wash rather than a glow. Raising the luminance threshold does not fix it, because the link colour is *brighter* in linear luma than the community node colours, so no threshold separates them. Upstream's bloom example pairs `strength: 4` with `backgroundColor="#000003"` for the same reason.

**Tuning knobs:** `BLOOM_STRENGTH` / `BLOOM_RADIUS` / `BLOOM_THRESHOLD` in `src/ForceGraph3DNetwork.tsx`. `BLOOM_RADIUS` controls the frame-wide wash most directly.

## 2026-08-22 — Brand Nodez + getnodez.app

- User-facing product name is **Nodez** (not Nodez).
- Public domain **getnodez.app** is the canonical site + OTA base.
- Vault meta directory is **`.nodez/`** (legacy **`.diamante/`** still readable). MCP env prefers `NODEZ_*` (legacy `DIAMANTE_*` accepted). Agent-contract markers accept both nodez and legacy diamante forms.
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
