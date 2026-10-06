---
id: nodez-ai-provider-updates
title: AI Provider Updates
type: workflow
status: active
created: 2026-10-02
updated: 2026-10-06
tags:
  - ai
  - providers
  - updates
---

# AI Provider Updates

Nodez Settings → AI Chat can check stable releases for Codex, Claude Code, Grok,
OpenCode, Gemini CLI, and ModelArk. Each provider reports its installed and
latest version independently, so one unavailable release service does not hide
the other results.

Updates require an explicit **Update** click and a confirmation showing the
exact command. Nodez blocks an update while that provider has a live session or
another update is running. It runs fixed commands directly, without a shell or
`sudo`, then verifies that the installed version changed.

Nodez updates recognized npm, Homebrew, or native CLI installations. A custom or
ambiguous executable gets **Show instructions** instead. Package-manager
permission problems remain visible for the user to resolve outside Nodez.

## Validation

- Provider version normalization and stable-release parsing have Rust tests.
- Allowlisted updater arguments and the global update lock have Rust tests.
- Frontend state, disabled actions, confirmation copy, and announcements have
  focused Node tests.
- TypeScript checks and the production Vite build pass.

Official references: [Codex CLI](https://developers.openai.com/codex/cli/),
[Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup),
[Grok CLI](https://github.com/xai-org/grok-cli),
[OpenCode](https://opencode.ai/docs/cli/),
[Gemini CLI](https://github.com/google-gemini/gemini-cli), and
[ModelArk CLI](https://docs.byteplus.com/en/docs/ModelArk/arkcli).

## October 6 fixes

Provider readiness and update commands run away from the desktop UI thread.
Installation checks resolve the executable path and follow symlinks to distinguish
npm packages from Homebrew installations. npm updates use the resolved Node
runtime, including Windows shims, so the desktop shell does not require a full
terminal PATH. Settings displays command failures and checking/updating progress.
A partial version increase leaves the provider marked as having an update available.

Reconnect after a provider update to discover its current model choices. All
providers remain listed in the switcher: previously discovered lists are cached,
and the original catalog is available before a provider's first successful
connection. A successful handshake replaces that provider's choices with its
full live list, without price or name/version restrictions. Model labels use IDs.
Codex includes hidden models and follows pagination; OpenCode supports paid and
grouped model choices; Claude keeps resolved IDs and context suffixes.

Model selection is scoped to the provider, workspace, and account. Other
providers no longer inherit the old global `nodez.codex.model` preference.
Restored selections absent from that provider's known choices are discarded;
new explicit model IDs saved under the provider's own preference remain valid
inputs. Exact ID entry is also available when a provider catalog is behind.

### October 6 verification

- All 404 script tests passed; native tests passed with 156 successful and one
  ignored billed ModelArk integration test.
- TypeScript lint, Markdown lint, and the macOS Tauri release build passed.
- Rebuilt and installed Nodez in `/Applications/Nodez.app` and verified its signature.
- Desktop smoke test: switching to OpenCode connected successfully without the
  stale Codex `gpt-5.6-sol` selection. Its menu listed paid and free models,
  including `opencode/gpt-6.1-sol`, alongside every provider's model choices.
- The full docs check still encounters the existing unsupported `proposed`
  status in [[Workspace Improvements Plan]]; that unrelated note was unchanged.

Windows desktop validation remains outstanding.

### npm installation prefix correction

The missing Codex `gpt-6.1-sol` was traced to the selected CLI still running
version 0.153.4. Its live model list omitted the newer model, while another Codex
installation's cache already contained it. npm's default global prefix differed
from the selected CLI's prefix, so a generic global update could install a new
copy without updating the executable Nodez actually launches.

npm update commands now derive `--prefix` from the selected executable's resolved
package path (or its Windows shim directory). The confirmation displays that
same argument. Updating the selected macOS installation produced Codex 0.160.1.
Regression coverage includes a symlink under a prefix containing spaces and
Windows shim placement; native validation passed with 158 tests and one ignored
billed integration test.

After reconnecting the installed app, the Codex selector visibly listed
`gpt-6.1-sol`, `gpt-6-sol`, and `gpt-6-luna`. No model aliases or extra UI filters
were needed; these IDs came directly from the updated CLI's `model/list`.
