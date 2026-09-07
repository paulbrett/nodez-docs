---
id: nodez-distribution-versioning-updates
title: Distribution Versioning and Updates
type: roadmap
status: active
created: 2026-08-21
updated: 2026-08-24
tags:
  - distribution
  - versioning
  - ota
  - tauri
  - roadmap
---

# Distribution Versioning and Updates

Plan for **app versioning**, **release artifacts**, and **OTA** so humans stay current without rebuilding from source. Complements [[Agent and Human Setup]] P7 and Phase 5 polish in [[Product Roadmap]].

Related: [[Next Steps]], [[Backlog]], [[Tauri Desktop Shell]], [[Landing Page]], [[GitHub Sync]]

## Current baseline (2026-08-22)

| Piece | State |
| --- | --- |
| `package.json` / tauri / Cargo version | `0.3.0` |
| Windows installers | MSI + NSIS via `npm run tauri -- build` |
| In-app About | Settings → About + **Check for updates** |
| Landing | `paulbrett/nodez` (public) → GitHub Pages |
| OTA manifest | landing repo `updates/latest.json` |
| Update bundles | landing repo `updates/bundles/` (CI) |
| Updater plugins | `tauri-plugin-updater` + `tauri-plugin-process` wired |
| Endpoint | `https://getnodez.app/updates/latest.json` |
| Public key | in `src-tauri/tauri.conf.json` `plugins.updater.pubkey` |
| Private key | **CI secret** `TAURI_SIGNING_PRIVATE_KEY`; local `src-tauri/nodez.key` (gitignored). Also `LANDING_DEPLOY_TOKEN` for CI push to landing |
| App icons | 1024 source `app-icon.png` → `npm run icons` → `src-tauri/icons/*` |
| Code signing (Authenticode) | not set up (later) |
| GitHub Releases | **v0.3.0** published (signed MSI/NSIS + sigs); landing Pages feed updated |

## Goals

1. **One version truth** — bump `package.json` + `tauri.conf.json` + `Cargo.toml` together.
2. **Human-safe upgrades** — consent via Settings; vault files never replaced by OTA.
3. **Reproducible releases** — tag → build → Release assets + Pages feed/bundles.
4. **Trust** — minisign updater signatures now; Authenticode later.

## Non-goals (v1)

- Auto-update without consent
- Silent background replace on every launch
- Updating vault / `.nodez` via OTA

## Work packages

### V1 — Versioning hygiene

- [x] Three-file version bump; Settings About
- [ ] `npm run version` bump script
- [x] Git tags `vMAJOR.MINOR.PATCH`
- [x] CHANGELOG per release
- [x] Semver sketch (patch / minor / major)

### V2 — Release pipeline

- [x] Landing Pages workflow (on `nodez`); app `release.yml`
- [x] `scripts/publish-update-feed.mjs` / `npm run update:feed`
- [x] Manifest + bundles on landing repo (`updates/`; CI force-adds gitignored binaries)
- [x] Signed `platforms.windows-x86_64` published to Pages (local signed build 2026-08-22)
- [x] Tag CI Release path (build + Release assets) verified **v0.3.0**; landing push fixed after first-run URL bug
- [ ] Authenticode for SmartScreen

### V3 — OTA (Tauri updater) — **landed skeleton 2026-08-22**

- [x] `@tauri-apps/plugin-updater` + Rust plugin
- [x] Endpoints → Pages static JSON
- [x] Public key in conf; private key in CI/local only
- [x] Settings → Check for updates → confirm → download/install → relaunch
- [ ] Optional launch-time quiet check + Remind later
- [x] Air-gap: check is manual; offline fails quietly with message
- [x] Vault untouched by updater

### V4 — Signing and trust

- [ ] Windows Authenticode
- [ ] macOS notarization when mac builds ship
- [ ] Checksums on [[Landing Page]]

## Acceptance

- [x] Landing + manifest path in repo
- [x] Live Pages feed has installers + signature (`latest.json` platforms filled)
- [x] Tagged app Release **v0.3.0** (assets on GitHub + feed on Pages)
- [ ] N-1 → N update path verified on a machine
- [x] Manual check only (no forced auto)
- [x] Vault survives update design

## Implementation pointers

| Piece | Location |
| --- | --- |
| Landing | `https://github.com/paulbrett/nodez` |
| Manifest | landing `updates/latest.json` |
| Feed script | `scripts/publish-update-feed.mjs` |
| Frontend | `src/appUpdate.ts`, Settings About in `App.tsx` |
| Config | `src-tauri/tauri.conf.json` `bundle.createUpdaterArtifacts`, `plugins.updater` |
| CI | landing `pages.yml`; app `release.yml` (needs `TAURI_SIGNING_PRIVATE_KEY` + `LANDING_DEPLOY_TOKEN`) |
| Local key | `src-tauri/nodez.key` — set `TAURI_SIGNING_PRIVATE_KEY` to **file contents** for `tauri build` |

## Secrets and local signing (2026-08-22)

| Secret (app repo `paulbrett/nodez-app`) | Purpose |
| --- | --- |
| `TAURI_SIGNING_PRIVATE_KEY` | Minisign private key **file contents** — signs updater artifacts in CI |
| `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` | Only if the key has a password (current key: empty) |
| `LANDING_DEPLOY_TOKEN` | Fine-grained PAT, **Contents: write** on `nodez` only — CI push of feed/bundles |

### Local PowerShell

```powershell
cd C:\Sites\nodez-app
$env:TAURI_SIGNING_PRIVATE_KEY = Get-Content -Raw .\src-tauri\nodez.key
$env:TAURI_SIGNING_PRIVATE_KEY_PASSWORD = ""
npm run tauri -- build
npm run update:feed
```

Note: `TAURI_SIGNING_PRIVATE_KEY_PATH` works for `tauri signer sign` but **`tauri build` requires `TAURI_SIGNING_PRIVATE_KEY` contents** on this toolchain.

### Git Bash

```bash
export TAURI_SIGNING_PRIVATE_KEY="$(cat src-tauri/nodez.key)"
export TAURI_SIGNING_PRIVATE_KEY_PASSWORD=""
npm run tauri -- build
```

Never commit `*.key`. Public key lives only in `tauri.conf.json` `plugins.updater.pubkey`.

## Status 2026-08-24 — CI release blocked on unset secrets

Release CI had **never** completed a run on either platform. 0.4.1's signed Windows
installer was therefore hand-built locally, not produced by the tag pipeline.

`gh api repos/paulbrett/nodez-app/actions/secrets` returns **`total_count: 0`** — none
of the three secrets above are actually set on the repo. An empty
`TAURI_SIGNING_PRIVATE_KEY` is what produces Tauri's misleading
`failed to decode secret key: ... Missing comment in secret key`.

`src-tauri/nodez.key` does **not** exist on the macOS machine either — the key lives on
the Windows box. Copy its full contents (including the leading `untrusted comment:`
line) into the repo secret to unblock CI.

Two workflow bugs were fixed at the same time, both of which had to be cleared before
the secrets even mattered:

- Windows failed at `Test`: a test asserted the POSIX literal `/vault/Folder/Note.md`,
  but `path.resolve("/vault")` is drive-relative on Windows (`D:\vault\...`).
- macOS failed at `Build Tauri`: `createUpdaterArtifacts` is on globally, so the mac
  build signed updater artifacts the feed can never publish (macOS stays download-only
  until codesign + notarytool). Mac now builds with them off and no signing env.

**macOS universal2 now builds green in CI.** Feed publishing stays blocked, which is
correct: `publish-update-feed.mjs` recomposes `latest.json` from same-version fragments
only, and macOS contributes no OTA entry — so a mac-only publish would emit
`"platforms": {}` and break Windows in-app updates.

### macOS build on this Mac — was a PATH problem (corrected 2026-08-24)

Earlier diagnosis said this Mac had no `rustup`. Wrong: `rustup` and both
`aarch64-apple-darwin` / `x86_64-apple-darwin` targets were already installed under
`~/.cargo`, but `~/.cargo/bin` was **not on PATH**, so Homebrew's **x86_64** rust at
`/usr/local/bin/cargo` shadowed it and `rustc -vV` reported host `x86_64-apple-darwin`.
That is why a universal build looked impossible.

Re-running rustup-init added `. "$HOME/.cargo/env"` to `.zshenv`, `.profile` and
`.bash_profile`, so new shells get the arm64 toolchain (rustc 1.98.0). Homebrew rust is
still installed — if `/usr/local/bin` ever precedes `~/.cargo/bin` again the shadowing
returns, so check `rustc -vV | grep host` before blaming the build.

Local universal build, with no signing key on this machine:

```bash
npm run tauri -- build --target universal-apple-darwin --bundles dmg,app \
  --config '{"bundle":{"createUpdaterArtifacts":false}}'
```

The `--config` override is needed for the same reason CI needed it: `createUpdaterArtifacts`
is on globally and macOS has no key and publishes no OTA entry.

### Resolved 2026-08-24 — bundled Node runtime removed; Node is a prerequisite

The bundled runtime was **never actually shipping**. `tauri.conf.json` packed it with the
shallow glob `resources/mcp/runtime/*`, which copies only top-level *files*, so
CHANGELOG/LICENSE/README made it into installers and `bin/node` never did. A 112MB binary
could not have fitted in a 10.2MB dmg. The "no-Node MCP kit" therefore always relied on the
user already having Node — on Windows and macOS alike, in 0.4.0 and 0.4.1.

Widening the glob to `runtime/**/*` fails the build outright:

```text
resource path `resources/mcp/runtime/bin/corepack` doesn't exist
failed to build aarch64-apple-darwin binary
```

`prepare-mcp-bundle` extracted the Node tarball to a temp directory and copied `bin/`
preserving symlinks, so `corepack`, `npm` and `npx` dangle into a path that no longer
exists. That is very likely why the shallow glob was chosen in the first place.

**Decision: do not bundle Node.** Beyond the above, a bundled Node is single-arch and so
cannot serve a universal2 app — Rosetta translates x86_64 to arm64, not the reverse, so an
Apple-Silicon-built runtime cannot execute on Intel at all.

Removed: the `runtime` resource entry, `--with-node`, `downloadNodeRuntime()`, `httpGet()`,
the `mcp:bundle:node` npm script, and the `withNode` marker in `bundle.json`. The generator
now also deletes any legacy `runtime/` it finds, and `.gitignore` still ignores that path so
an old checkout's copy can never be committed.

**In its place:** the launchers resolve Node as `NODEZ_NODE_COMMAND` → PATH → common install
locations (Homebrew both prefixes, `/usr/bin`, Volta, asdf, fnm, highest nvm version;
`Program Files\nodejs`, LocalAppData, Volta, nvm on Windows), then print install guidance.
The fallback is the important part: an MCP client launched from Finder or the Start menu
inherits a reduced PATH that routinely omits Node. Verified live — with PATH stripped of
Node, the launcher still resolved `/usr/local/bin/node` and served the vault.

The app itself now runs `node --version` rather than assuming `"node"` works, and
Settings → Agents shows the detected version or per-platform install instructions.

### `src-tauri/resources/mcp/` is generated output

`beforeBuildCommand` runs `prepare-mcp-bundle.mjs` on every build, and it rewrites the
launchers from inline templates. Editing those files directly appears to work and is
silently reverted at the next build — **edit the generator**. They are still tracked in git
because `tauri dev` copies them as-is without running the generator, which is why their
committed file mode matters for dev builds (and only for dev).

### Verified 0.4.2 macOS build

- `lipo -archs` on the app binary: `x86_64 arm64` (genuine universal2)
- `Contents/Resources/resources/mcp/` contains the scripts, launchers and `node_modules/yaml`
- **no `runtime/` directory**
- `nodez-mcp.sh` is `-rwxr-xr-x`
