---
id: markdown-health-and-repair
title: Markdown Health and Repair
type: workflow
status: active
created: 2026-09-10
updated: 2026-09-10
tags:
  - markdown
  - vault
  - maintenance
  - editor
---

# Markdown Health and Repair

Nodez now has a vault-wide Markdown audit and conservative repair workflow. It
belongs in **Settings → Vault** because it acts on the active vault rather than
one note, one repository, or an agent session. Repeat users can also launch
**Audit and repair Markdown** from the command palette.

## Workflow

1. **Scan** — inspect every loaded Markdown note in small UI-thread chunks with a
   determinate progress indicator.
2. **Review** — group findings by note and show Healthy, Safe fix, Review, and
   Manual totals.
3. **Apply** — select safe changes only after viewing a per-note before/after
   preview.
4. **Results** — report applied, changed-since-scan, and failed files separately.
   A one-step undo remains available until the dialog closes.

The scanner never launches changes automatically.

## Interface behavior

The dialog is centered in the active window. Scanning and applying use a staged,
determinate loader. Findings use the larger split review workspace; a clean scan
contracts to a compact result instead of leaving an empty full-size canvas. The
step row has explicit marker geometry and vertical rhythm, and connector lines
are removed on very narrow windows where they would collide with labels. A clean
result offers **Done** rather than a disabled Apply action.

## Safe automatic repairs

The first release limits automatic changes to syntax-preserving cleanup:

- remove a leading byte-order mark;
- clear whitespace on otherwise blank lines;
- collapse excessive blank-line runs outside fenced code;
- remove trailing blank lines;
- add one final newline.

Fenced code contents and Markdown two-space hard breaks are preserved.

## Review-only findings

Nodez reports but does not rewrite:

- prose that may need Markdown structure;
- malformed heading candidates;
- unclosed YAML frontmatter;
- unclosed fenced code blocks;
- unresolved wikilinks;
- unsupported Obsidian-style embeds.

This boundary prevents a vault-wide maintenance action from guessing at meaning,
renaming link targets, or adopting attachment behavior before portable vault
attachments land.

## Write safety

Desktop repairs use a revision-checked native command. Each write succeeds only
when the on-disk note still exactly matches the content captured by the scan. The
replacement is written to a temporary file in the same directory, synchronized,
checked again, and atomically persisted. Externally changed notes are skipped and
reported instead of overwritten.

Browser-demo notes use the existing local storage path.

## Implementation

- Scanner and normalizer:
  `src/features/notes/markdownHealth.ts`
- Review and progress dialog:
  `src/features/notes/MarkdownHealthDialog.tsx`
- Settings and command-palette integration:
  `src/App.tsx`, `src/shared/commands.ts`
- Revision-checked write bridge:
  `src/shared/vault.ts`, `src-tauri/src/lib.rs`
- Focused tests:
  `scripts/markdown-health.test.mjs`

## Validation

Completed on 2026-09-10:

- 15 focused Markdown preview, note-link, and health tests passed.
- TypeScript, Markdown lint, and docs frontmatter checks passed.
- 89 Rust library tests passed.
- Production build passed after the final normalizer safety update.
- The complete JavaScript test suite passed: 264 tests.
- A read-only scan of the current 48-note docs vault found no repair candidates.

A live packaged desktop visual smoke test remains outstanding. Validate the dialog
in a large vault, an externally edited note conflict, keyboard-only operation,
and a compact window before treating the surface as release-certified.

Related: [[Next Steps]], [[Unified Knowledge System]],
[[Command Palette and Agent Surface]], [[Agent Skills and Surfaces]]
