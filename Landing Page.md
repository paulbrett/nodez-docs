---
id: diamante-landing-page
title: Landing Page
type: backlog
status: active
created: 2026-08-21
updated: 2026-08-21
tags:
  - landing
  - marketing
  - distribution
  - backlog
---

# Landing Page

TODO and scope for a **public Diamante marketing / download site**. Not the app UI and not the docs vault — a simple web presence so humans can understand the product and get installers (and later OTA feed links).

Related: [[Distribution Versioning and Updates]], [[Product Roadmap]], [[Project Overview]], [[Agent and Human Setup]], [[Next Steps]]

## Why

- Installers exist (Windows MSI/NSIS) but discovery is "know the repo"
- OTA and versioning need a stable public URL for downloads and release notes
- Explains local-first + vault + graph + agent/MCP story in one scroll

## Non-goals (v1)

- Full docs site (docs stay in this Obsidian vault / later docs subdomain)
- In-browser full Diamante app
- Accounts, cloud sync signup, or waitlist backend (optional later)
- Replacing GitHub Releases as artifact storage (landing can deep-link)

## Placement

| Option | Notes |
| --- | --- |
| A. Static site in monorepo (`landing/` or `www/`) | Simple; deploy to GitHub Pages / Cloudflare Pages |
| B. Separate repo | Cleaner if marketing iterates faster than app |
| C. Subpath on existing personal site | Fastest if one already exists |

**Default recommendation:** A — `landing/` static (HTML or light Vite) in `C:\Sites\diamante`, deploy on tag/release. Revisit B if content freelancing diverges.

## v1 content checklist

### Hero

- [ ] Product name + one-line thesis (local Markdown vault + project graph + agent-ready)
- [ ] Primary CTA: **Download for Windows**
- [ ] Secondary CTA: **View on GitHub** / docs pointer
- [ ] Optional: short loop video or still of graph + editor (no stock AI sludge)

### Value props (3–5)

- [ ] Plain files, Obsidian-compatible vault habits
- [ ] Repo + notes in one graph
- [ ] Local-first; GitHub sync direction (honest: sync preview / roadmap if not shipped)
- [ ] MCP / AI agent surface (query graph, edit notes)
- [ ] Open workspace you own

### Download

- [ ] Latest version number (from release API or build-time inject)
- [ ] MSI and/or NSIS links
- [ ] Checksums
- [ ] System requirements (Windows 10/11, WebView2)
- [ ] macOS / Linux: "coming soon" or hide until real

### Agent / MCP blurb

- [ ] Short "works with Hermes / Claude / Cursor via MCP" with link to setup note once public
- [ ] Do not require Node in the happy path copy once no-Node MCP ships — until then be honest

### Trust / local-first

- [ ] No proprietary cloud required for core editing
- [ ] Vault stays on disk; graph artifact is generated index

### Footer

- [ ] License
- [ ] GitHub
- [ ] Contact / author
- [ ] Link to release notes / changelog

## v1.1 polish

- [ ] Dark theme matching app Graphite/Dark tokens (restrained; see vault [[Frontend]] taste notes if reused)
- [ ] Responsive mobile layout
- [ ] OG/twitter meta image
- [ ] Analytics only if privacy-acceptable (prefer none or privacy-first)
- [ ] `sitemap.xml` / canonical URL
- [ ] Deep link: `/download` and `/releases`

## Engineering TODO

- [ ] Scaffold `landing/` (or chosen option) with minimal deps
- [ ] Deploy workflow (Pages/Cloudflare) on `main` or release tag
- [ ] Wire download buttons to latest GitHub Release assets
- [ ] Optional: fetch `latest` release JSON for version label
- [ ] Add landing URL to app About and README
- [ ] Add landing URL to [[Distribution Versioning and Updates]] updater endpoint decision if feed is hosted here
- [ ] Keep marketing claims aligned with [[Product Roadmap]] (no vapor features)

## Copy constraints

- Prefer concrete verbs: open vault, attach repo, query graph, export MCP config
- Avoid "AI-powered second brain" clichés
- Prefer "agents use MCP tools" over "autonomous AGI workspace"
- Sync: say "Git-friendly / GitHub-oriented" until Phase 2 ships real pull/push

## Acceptance (landing v1)

- [ ] Public URL loads in <3s on broadband, readable offline-cached static HTML
- [ ] Windows user can reach an installer in ≤2 clicks from hero
- [ ] Version on page matches latest published release
- [ ] No broken claims vs shipped app on that release tag

## Open decisions

1. Domain name?
2. Hosting (GitHub Pages vs Cloudflare vs other)?
3. Feed host for OTA: GitHub Releases direct vs landing-proxied manifest?
4. Ship landing before or with first signed public build?

Capture answers in [[Decision Log]] when chosen.
