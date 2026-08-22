---
id: diamante-landing-page
title: Landing Page
type: roadmap
status: active
created: 2026-08-21
updated: 2026-08-22
tags:
  - landing
  - marketing
  - distribution
  - roadmap
---

# Landing Page

Public Diamante marketing / download site. **v1 shipped in-app repo** as plain HTML/CSS.

Related: [[Distribution Versioning and Updates]], [[Product Roadmap]], [[Project Overview]], [[Agent and Human Setup]], [[Next Steps]]

## Placement (decided 2026-08-22)

| Choice | Detail |
| --- | --- |
| Location | `C:\Sites\diamante\landing\` in main app repo |
| Stack | Single-page **plain HTML + CSS** (tiny JS only to soft-fill version from manifest) |
| Host | GitHub Pages — `https://paulbrett.github.io/diamante/` |
| OTA | Manifest + bundles under `landing/updates/` |

```text
landing/
  index.html
  styles.css
  updates/
    latest.json     # Tauri updater feed + installers map
    bundles/        # MSI/NSIS + signed updater artifacts (CI; not committed)
    README.md
```

## v1 content checklist

### Hero

- [x] Product name + one-line thesis
- [x] Primary CTA: **Download for Windows**
- [x] Secondary CTA: **View on GitHub**
- [ ] Optional loop video / product still

### Value props

- [x] Plain files / vault habits
- [x] Repo + notes graph
- [x] Local-first; GitHub vault sync (honest: shipped v1)
- [x] MCP / external agents
- [x] Workspace you own

### Download

- [x] Version from `updates/latest.json`
- [x] MSI + NSIS links (relative under `updates/bundles/` + Releases)
- [ ] Checksums on page
- [x] System requirements (Win10/11, WebView2)
- [x] macOS / Linux: not shipped yet

### Agent / MCP blurb

- [x] Hermes / Claude / Cursor via MCP; Node-based until no-Node path

### Footer

- [x] GitHub, changelog, latest.json, issues

## Engineering

- [x] Scaffold `landing/`
- [x] Pages workflow (`.github/workflows/pages.yml`)
- [x] Release workflow copies feed + bundles (`.github/workflows/release.yml`)
- [x] `scripts/publish-update-feed.mjs` + `npm run update:feed`
- [x] App Settings → About links site + Check for updates
- [x] Updater endpoint → Pages `updates/latest.json`

## Acceptance

- [ ] Public URL live after first Pages enable + push
- [x] ≤2 clicks to installer once bundles published
- [ ] Version on page matches published signed release
- [x] Claims match shipped features (sync v1, MCP Node)

## Open / next

1. Enable GitHub Pages (source: GitHub Actions) on `paulbrett/diamante`
2. Add `TAURI_SIGNING_PRIVATE_KEY` repo secret; run tag release so `platforms.windows-x86_64` is signed
3. Authenticode later; checksums on page
4. Custom domain optional
