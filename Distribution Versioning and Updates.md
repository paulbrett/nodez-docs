---
id: nodez-distribution-versioning-updates
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
