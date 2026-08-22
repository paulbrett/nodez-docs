---
id: diamante-distribution-versioning-updates
title: Distribution Versioning and Updates
type: roadmap
status: active
created: 2026-08-21
updated: 2026-08-21
tags:
  - distribution
  - versioning
  - ota
  - tauri
  - roadmap
---

# Distribution Versioning and Updates

Plan for **app versioning**, **release artifacts**, and **OTA (over-the-air) updates** so humans stay on a current Diamante without reinstalling from source. Complements [[Agent and Human Setup]] P7 and Phase 5 polish in [[Product Roadmap]].

Related: [[Next Steps]], [[Backlog]], [[Tauri Desktop Shell]], [[Landing Page]], [[GitHub Sync]]

## Current baseline (2026-08-21)

| Piece | State |
| --- | --- |
| `package.json` version | `0.2.0` |
| `src-tauri/tauri.conf.json` version | `0.2.0` |
| `src-tauri/Cargo.toml` version | `0.2.0` |
| Windows installers | MSI + NSIS via `npm run tauri -- build` |
| In-app About version | Settings → About shows `v{package.json version}` |
| Code signing | not set up |
| Tauri updater plugin | not configured |
| Public download / release feed | GitHub Releases for tagged builds (starting `v0.2.0`) |

## Goals

1. **One version truth** — bump once; npm, Tauri, installers, About UI, and release tags agree.
2. **Human-safe upgrades** — installed app checks for updates and can apply them with consent (OTA).
3. **Reproducible releases** — tag → build → publish artifacts + update manifest.
4. **Trust** — signed installers/updates on Windows (and later macOS notarization).

## Non-goals (v1 of this track)

- Auto-update without user consent
- Silent background binary replacement on every launch
- Enterprise MDM / custom private update channels (later)
- Updating vault data or graph artifacts via OTA (app binary only; vault stays user files)

## Work packages

### V1 — Versioning hygiene

- [x] Three-file version bump (`package.json` + `tauri.conf.json` + `Cargo.toml`) documented; Settings About reads `package.json`
- [ ] `npm run version` / release script bumps package + `tauri.conf.json` (+ lockfile if needed) automatically
- [x] Show version in Settings / About
- [x] Git tags `vMAJOR.MINOR.PATCH` match shipped binaries (from `v0.2.0`)
- [x] CHANGELOG in app repo per release
- [x] Document semver policy: breaking vault meta / MCP tool renames = minor or major intentionally

#### Semver sketch for Diamante

| Bump | When |
| --- | --- |
| patch | bugfix, UI polish, no MCP/schema break |
| minor | new MCP tools, new graph features, backward-compatible meta |
| major | vault meta or MCP breaking changes, forced re-index, security model change |

### V2 — Release pipeline

- [ ] GitHub Release (or equivalent) per tag with MSI + NSIS (+ later macOS/Linux)
- [ ] Generate Tauri updater JSON/manifest alongside artifacts
- [ ] CI: `npm run check` + `tauri build` on tag push (Windows runner first)
- [ ] Keep unsigned local builds for dev; signed builds for public channel only
- [ ] Release checklist in vault or app `docs/` (preflight: lint, check, smoke vault open, MCP smoke)

### V3 — OTA updates (Tauri updater)

Prefer official **Tauri 2 updater** plugin pattern:

- [ ] Add `@tauri-apps/plugin-updater` (+ rust plugin)
- [ ] Configure `plugins.updater.endpoints` to a static HTTPS feed (GitHub Releases CDN or project site)
- [ ] Public key for update signature verification; private key only in CI secrets
- [ ] On launch or via Settings "Check for updates": fetch → show notes → download → install → relaunch
- [ ] Respect "remind me later" / disable check for air-gapped users
- [ ] Never clobber open vault files; updater replaces app install only
- [ ] Windows: NSIS/MSI strategy documented (which artifact the updater consumes)

#### Failure modes to design for

- Offline / feed unreachable → quiet fail, no modal spam
- Signature mismatch → hard fail, no install
- Partial download → retry cleanly
- User mid-edit → prompt to save / confirm quit before relaunch

### V4 — Signing and trust

- [ ] Windows Authenticode signing for installers and update payloads
- [ ] macOS signing + notarization when mac builds ship
- [ ] Publish checksums on [[Landing Page]] download section
- [ ] Security note: MCP server path and update feed are separate trust surfaces

## Sequencing vs other tracks

| Order | Rationale |
| --- | --- |
| After usable MSI/NSIS (done) | OTA needs something to update from |
| Parallel with [[Landing Page]] | Landing hosts download + link to latest release / feed |
| After or with P2 human setup | Version + update UX belongs in Settings next to MCP export |
| Not blocking P0–P3 agent track | Agents do not need OTA; humans do for retention |

Suggested placement in overall roadmap: **Phase 5 / P7 distribution**, with V1 versioning done early (cheap) and V3 OTA when public downloads exist.

## Acceptance checks

- [ ] Bumping version in one place updates installer metadata and About UI
- [ ] Tagged release publishes installers + updater manifest
- [ ] Fresh install of N-1 offers update to N, verifies signature, relaunches on N
- [ ] Air-gapped machine can disable update checks and keep working
- [ ] Vault and `.diamante/` survive update unchanged

## Implementation pointers (app repo)

| Piece | Likely location |
| --- | --- |
| Version | `package.json`, `src-tauri/tauri.conf.json` |
| About UI | Settings modal / future About command in palette |
| Updater | Tauri 2 plugin updater + endpoint JSON |
| CI | `.github/workflows/release.yml` (to add) |
| Feed host | GitHub Releases and/or landing site |

App/code: `C:\Sites\diamante`. Docs vault: `C:\Users\webwi\Documents\Diamante`.
