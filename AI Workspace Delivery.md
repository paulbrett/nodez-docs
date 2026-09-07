---
id: nodez-ai-workspace-delivery
title: AI Workspace Delivery
type: architecture
status: active
created: 2026-09-07
updated: 2026-09-07
tags:
  - codex
  - editor
  - release
---

# AI Workspace Delivery

Release candidate: **0.6.0-rc.1**. Implementation follows the user's approved
sequence in [[Code Editor and AI Chat Plan]] and [[Codex Chat Implementation]].
Full access was explicitly requested on 2026-09-07; it supersedes the original
fixed read-only runtime configuration, while repository editor access remains a
separate capability.

## Delivered in source

- [x] Prominent Notes/Code and AI Chat workspace tabs (updated from the original
  right-side panel at user request). Chat uses the full editor area; tab switching
  preserves the conversation and composer draft.
- [x] Read-only, Ask for approval, and Full access modes fixed per connection.
- [x] Window/session-scoped native App Server bridge, account and model discovery,
  streaming, command activity, diffs, one-time approvals, questions and Stop.
- [x] Foreign-thread and obsolete-turn events rejected; completed turns cannot
  resurrect late output. Closing/switching ends the native session.
- [x] Repository file tree, retained editor tabs, dirty indicators, explicit Save
  and Reload, editing disabled until enabled, and unsaved-discard confirmations.
- [x] Native editor access separate from Codex and source indexing. Relative paths
  only, symlinks and metadata paths rejected, UTF-8 text limited to 1 MB.
- [x] SHA-256 disk revision checks before and immediately prior to atomic replacement;
  file permissions preserved. A conflict preserves the user's draft.
- [x] External-change checks every three seconds and on focus, without re-indexing.
- [x] Single-file structured AI proposals bind file, UTF-16 range and buffer hash;
  before/after review, Apply to buffer and Discard. Apply checks disk revision and
  current buffer hash and is isolated as one CodeMirror undo action. Save is separate.
- [x] Current-file and selection context, current-note snapshot and graph-neighbor
  opt-ins with previews. Graph relations retain provenance, confidence and source.
- [x] Known file citations resolve to editor lines. Known note citations open notes;
  missing references report a clear error. Dirty/indexing graphs are flagged.
- [x] Optional workspace-local display transcripts in `.nodez/chat-history`, with
  view/delete controls and a nested `.gitignore`. Off by default. JSON transcripts
  are outside note listing and curated wiki publishing. Codex retains its own history.
- [x] Close returns focus to the chat toolbar button. File/output limits and bounded
  transcript rendering; editor limits open files to 20 and tree results to 500.

## Validation evidence

- Live macOS desktop: Connect, existing authentication and model list worked.
- Live Codex turn: streamed commentary, `pwd` command activity with exit 0, final
  `NODEZ_CHAT_OK`, then a second turn exercised Stop.
- Controlled desktop protocol fixture: file diff and approval card rendered,
  Allow once was returned through native IPC, question selection was submitted,
  and `FIXTURE_OK` rendered with no pending requests. Fixture executes no commands
  and writes no files. Script: `scripts/codex-protocol-fixture.mjs`.
- macOS editor: source tree and editor opened, enabling editing and typing showed
  the dirty indicator, Undo cleared it. The test did not save changes to app source.
- 126 JavaScript tests passed, including proposal staleness, undo separation,
  context bounds, citations, streaming, obsolete events and existing regressions.
- 24 Rust tests passed, including atomic-save conflicts, path/symlink containment,
  file bounds, history scoping/deletion, native response routing and approvals.
- TypeScript, docs checks and production frontend build passed.

## Remaining release gates and limits

- [ ] Windows desktop smoke: connect/login, Stop, approval interaction, file saving,
  close-with-unsaved handling and installer execution require a Windows host.
- [x] Packaged macOS desktop end-to-end with real Codex: proposal → Review →
  Apply (disk unchanged) → explicit Save (reviewed text on disk). External-change
  conflict UI rejected Save, preserved the external file and retained the draft.
  Reload confirmation worked; temporary scratch file was removed afterward.
- [ ] Fresh device-code login is untested because this Mac already has Codex auth.
- [x] Packaged macOS app launched and passed the editor/Codex checks above.
  Apple Silicon `.app` and `.dmg` built under `src-tauri/target/release/bundle/`.
  The DMG installation flow itself has not been exercised.
- [ ] Release publication, signing/notarization and update-feed publication.

Atomic rename prevents partial writes. Revision checks detect ordinary external
edits; no portable filesystem compare-and-swap can eliminate a write by an
uncooperative external process in the final check-to-rename interval. Symlink
checks cover ordinary paths, not an adversarial process swapping ancestor
directories concurrently. Do not describe this editor as a hardened filesystem
sandbox against another process running as the same user.

Full access grants runtime writes and commands beyond the workspace without
approval prompts. Commit/push/publish/delete restrictions remain assistant
instructions in that mode, not OS-enforced restrictions. MCP tool exposure still
contains only the active workspace's read tools.

Local saved transcripts are display-only, not thread resumption. Multi-file AI
patch application, arbitrary new source-file creation, language-server integration
and cross-platform release certification remain follow-on work.

## Chat layout refinement — 2026-09-07

The user supplied a VS Code Codex chat reference and requested the same general
chat layout while preserving Nodez styling. AI Chat now has a prominent workspace
tab outside the Markdown toolbar. Notes and code stay mounted when chat is active,
and citations/proposal review switch back to the editor tab. Arrow keys navigate
the workspace tabs; explicit close still disconnects the Codex session.

Chat renders Markdown with highlighted fenced code and inline code, using the
existing note-preview syntax colors. Code blocks expose Copy. Unknown languages,
large blocks and incomplete streamed fences fall back to readable text. Raw HTML
is escaped, remote images are not loaded, and citations remain scoped to Nodez.

The composer places context, permissions, model selection and Send/Stop along its
bottom edge. Supporting connection/history controls are collapsed above the
transcript. Full-access implications remain visible when selected. The previous
macOS installer predates this UI refinement; the running development app contains
these changes until the next package build.

UI refinement validation: 12 focused Markdown/chat/context tests passed, including
highlighted TS, partial fences and raw-HTML escaping. TypeScript and frontend build
passed. Live macOS Codex returned a highlighted TypeScript block with Copy controls;
switching Notes → AI Chat preserved the connected session and response. Notes
retain their existing editor and inspector layout.

### Shared explorer refinement

- AI Chat and repository editing reuse the left sidebar for repository files, with a separate search filter. Back to notes restores the notes explorer.
- Removed the duplicate file tree inside the repository editor to give code more space.
- Graph is available at the top-right of both workspace tabs; removed the Notes inspector graph card and old explorer Graph shortcuts.
- Verified TypeScript, production build, and native desktop switching from chat explorer to an open source file. Source editing remains explicitly opt-in.

### Chat and explorer visual polish

- Added compact folder/file icon rows, open-folder states, alphabetical grouping, and theme-aware selection/focus states.
- Added chat starter prompts and a contextual empty state; refined composer focus and spacing without changing the app palette or typography.
- Fixed missing CodeMirror syntax color extension; repository files now load additional language grammars by filename on demand. Unknown formats retain plain text, and read-only buffers also enforce CodeMirror's read-only state.
- TypeScript, production build, and six editor/chat Markdown tests pass. Confirmed syntax colors in the native file editor.

### Image submission follow-through

- Image-only messages now enable Send; attached PNG/JPEG/WebP/GIF images use Codex image inputs with previews, removal, and clipboard paste.
- Up to four images, 8 MB each. Native validation rejects unsupported data URLs, malformed base64, and oversized payloads.
- Pending reads cannot restore images after disconnect/unmount, and sending waits for image reads. Image payloads are excluded from saved transcripts.
- TypeScript, production build, 26 Rust tests, and seven chat tests pass. Live image response remains to be verified.
