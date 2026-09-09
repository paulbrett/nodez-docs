---
id: nodez-september-9-improvements
title: September 9 Improvements
type: architecture
status: active
created: 2026-09-09
updated: 2026-09-09
tags:
  - acceptance
  - search
  - reliability
---

# September 9 Improvements

Accepted sequence: provider reliability and editing acceptance, release validation,
then ranked note search. Attachments and graph freshness/navigation remain the next
feature slices. This note supersedes conflicting historical backlog status; it does
not certify a release.

## Source checkpoint

App changes are committed on `main` as
`c9a9b26850134c35443fcb6eb104d81c759a64f6` — `feat: improve note search and workspace reliability`.
The user authorized pushing the app and docs repositories. This is a source/docs
checkpoint, not a tagged release or completed platform certification.

Final pre-commit checks: 200 JavaScript tests, 86 Rust tests, TypeScript/docs
checks, production frontend build, and diff whitespace checks passed.

## Changes

- Ranked note search: exact title, title prefix, partial title, path, then body.
  Flat results avoid folder sorting overriding relevance. Results include bounded
  context, literal highlights, and matching line numbers. Down from search focuses
  results; arrows navigate; Enter opens in Markdown at the matching line; Escape
  in the search field restores the tree. The UI displays the first 100 results.
- Malformed stored keys now return an error instead of panicking on a Unicode
  byte boundary. Empty encoded keys are rejected. Existing local credential
  storage changes were preserved.
- Reviewed edits revalidate all buffer contents and revisions after asynchronous
  checks. A changed or closed buffer rejects the whole Apply all operation and
  leaves the proposal available. Obsolete proposal/session results cannot apply.
- Carried provider text is bound to its originating vault/repository before a new
  panel mounts. Previously an effect cleared it too late, allowing old text into
  the new workspace. Starting a new conversation or selecting a saved one clears
  pending provider carry text. The composer label is now provider-neutral.

## Validation

- 200 JavaScript tests passed, including ranking, a 1,000-note vault, literal
  highlights, malformed proposals, undo isolation, stale multi-file buffers, and
  workspace-scoped carried text.
- Native malformed-key regression reproduced a panic before the fix and passed
  afterward. All 86 Rust tests passed.
- Live macOS development app: ranked search, body snippets, arrow navigation,
  Enter/click opening, and Markdown focus verified in temporary vaults.
- Live scratch repository: typing kept disk unchanged; explicit Save wrote the
  buffer; a later external edit rejected Save and preserved both disk content and
  draft. Cancelling the unsaved-close dialog retained the draft.
- Fresh-vault chat started without old transcript text after the scope fix.
- Starting a new OpenCode conversation cleared pending carry text before sending.
- macOS application and Apple Silicon DMG builds passed. The freshly packaged
  application launched with its onboarding/gateway UI. The DMG installation flow
  and installed upgrade have not been exercised.

## Provider acceptance on this Mac

| Provider | Installed CLI | Live evidence | Remaining |
| --- | --- | --- | --- |
| Codex | 0.147.0 | Connected; exact smoke response | Complete permission/approval/Stop matrix |
| Claude | 2.1.233 | Connected with Haiku; exact smoke response | Stop and both supported modes |
| Grok | 1.0.13 | Connected; exact smoke response; streaming stopped | Read-only and approval interactions |
| OpenCode | 1.18.23 | Connected; model discovered; reconnect and exact smoke response | Full permission matrix |
| Gemini | 0.59.0 | CLI detected; missing key reported | User must supply a Gemini API key in Settings → Keys |

## Remaining release gates

- Finish the provider matrix, including approvals, images where supported, and
  cancellation/reconnect across all supported modes. Basic prompting is not full
  provider acceptance.
- Windows desktop/installer smoke requires a Windows host.
- Installed upgrade from N−1 to N, signed update feed, signing/notarization, and
  release publication remain unverified. No release has been published here.
- Attachments and graph freshness/source navigation are not implemented in this
  cycle. Node.js 20+ remains required for MCP.

Related: [[Next Steps]], [[Backlog]], [[AI Workspace Delivery]],
[[AI Chat Follow-on Phases]], [[Gemini CLI Provider]].
