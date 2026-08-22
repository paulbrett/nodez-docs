---
id: nodez-page
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

Public Nodez marketing / download site. **v1 shipped in-app repo** as plain HTML/CSS.

Related: [[Distribution Versioning and Updates]], [[Product Roadmap]], [[Project Overview]], [[Agent and Human Setup]], [[Next Steps]]

## Placement (decided 2026-08-22)

| Choice | Detail |
| --- | --- |
| Location | Separate public repo `paulbrett/nodez` → `C:\Sites\nodez` |
| Stack | Single-page **plain HTML + CSS** (tiny JS only to soft-fill version from manifest) |
| Host | GitHub Pages — `https://getnodez.app/` |
| OTA | Manifest + bundles under landing repo `updates/` |

```text
nodez/          # github.com/paulbrett/nodez (public)
  index.html
  styles.css
  updates/
    latest.json
    bundles/
  .github/workflows/pages.yml
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

- [x] Scaffold `nodez` repo (split from app)
- [x] Pages workflow on landing repo
- [x] App release workflow can push feed + bundles into landing repo
- [x] `scripts/publish-update-feed.mjs` + `npm run update:feed`
- [x] App Settings → About links site + Check for updates
- [x] Updater endpoint → `https://getnodez.app/updates/latest.json`

## Acceptance

- [x] Public URL live: <https://getnodez.app/>
- [x] ≤2 clicks to installer (signed 0.3.0 NSIS/MSI on Pages)
- [x] Feed version 0.3.0 + `platforms.windows-x86_64` signed
- [x] Claims match shipped features (sync v1, MCP Node)

## Open / next

1. Tag app Release via CI to dual-publish GitHub Release assets (Pages already has feed)
2. Authenticode later; checksums on page
3. Custom domain optional
