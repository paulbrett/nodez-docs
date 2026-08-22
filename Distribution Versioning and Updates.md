---
id: diamante-distribution-versioning-updates
title: Distribution Versioning and Updates
type: roadmap
status: active
created: 2026-08-21
updated: 2026-08-22
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
| Landing | `landing/` plain HTML/CSS → GitHub Pages |
| OTA manifest | `landing/updates/latest.json` |
| Update bundles | `landing/updates/bundles/` (CI; gitignored binaries) |
| Updater plugins | `tauri-plugin-updater` + `tauri-plugin-process` wired |
| Endpoint | `https://paulbrett.github.io/diamante/updates/latest.json` |
| Public key | in `src-tauri/tauri.conf.json` `plugins.updater.pubkey` |
| Private key | **CI secret** `TAURI_SIGNING_PRIVATE_KEY` (local: `~/.tauri/diamante.key`) |
| Code signing (Authenticode) | not set up |
| GitHub Releases | `v0.2.0` published; `v0.3.0` when tagged |

## Goals

1. **One version truth** — bump `package.json` + `tauri.conf.json` + `Cargo.toml` together.
2. **Human-safe upgrades** — consent via Settings; vault files never replaced by OTA.
3. **Reproducible releases** — tag → build → Release assets + Pages feed/bundles.
4. **Trust** — minisign updater signatures now; Authenticode later.

## Non-goals (v1)

- Auto-update without consent
- Silent background replace on every launch
- Updating vault / `.diamante` via OTA

## Work packages

### V1 — Versioning hygiene

- [x] Three-file version bump; Settings About
- [ ] `npm run version` bump script
- [x] Git tags `vMAJOR.MINOR.PATCH`
- [x] CHANGELOG per release
- [x] Semver sketch (patch / minor / major)

### V2 — Release pipeline

- [x] Workflows: `.github/workflows/pages.yml`, `release.yml`
- [x] `scripts/publish-update-feed.mjs` / `npm run update:feed`
- [x] Manifest + bundles co-located under `landing/updates/`
- [ ] First signed tag release with non-empty `platforms.windows-x86_64`
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
- [ ] Tagged release publishes installers + signed updater payload + updated `latest.json`
- [ ] N-1 → N update path verified on a machine
- [x] Manual check only (no forced auto)
- [x] Vault survives update design

## Implementation pointers

| Piece | Location |
| --- | --- |
| Landing | `landing/` |
| Manifest | `landing/updates/latest.json` |
| Feed script | `scripts/publish-update-feed.mjs` |
| Frontend | `src/appUpdate.ts`, Settings About in `App.tsx` |
| Config | `src-tauri/tauri.conf.json` `bundle.createUpdaterArtifacts`, `plugins.updater` |
| CI | `.github/workflows/pages.yml`, `release.yml` |
