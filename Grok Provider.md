---
id: nodez-grok-provider
title: Grok Provider
type: architecture
status: active
created: 2026-09-08
updated: 2026-09-08
tags:
  - grok
  - agents
  - planning
---

# Grok Provider

Implemented on the feature branch on 2026-09-08. Grok joins Codex and
[[Claude Code Provider]] as a third agent behind the same chat surface, over the
Agent Client Protocol.

Spec: `docs/superpowers/specs/2026-09-08-grok-provider-design.md`.
Protocol captures: `docs/superpowers/plans/2026-09-08-grok-protocol-findings.md`.
Implementation record: `docs/superpowers/plans/2026-09-08-grok-provider.md`.

## Why ACP rather than the simpler path

Grok Build offers two transports. `--output-format streaming-messages-json`
emits the Anthropic Messages wire format, which the Claude translator already
parses almost verbatim — the cheap option. `grok agent stdio` speaks ACP, a
JSON-RPC protocol needing its own translator.

ACP won because the cheap path cannot scope MCP to a vault, and it spawns a
process per turn. Reuse was not worth breaking vault isolation.

## Decisions — 2026-09-08

The user chose each of these.

- **ACP transport** (`grok agent stdio`), accepting a second translator.
- **Parity with Claude plus approvals.** Grok is the first provider that can
  offer all three permission modes.
- **An xAI API key stored by Nodez**, in the OS credential store via the
  `keyring` crate. This **supersedes the "no new provider API key storage"
  constraint** recorded in [[Code Editor and AI Chat Plan]]: under the isolation
  mechanism below, Grok cannot see a `grok login` credential, so an API key is
  the only verified way to authenticate an isolated session.

## What the spike established

Verified against Grok Build 1.0.13. The CLI auto-updates, so these should be
re-checked rather than trusted indefinitely.

**Isolation uses a Nodez-controlled `HOME`.** Grok discovers MCP servers from
other tools' configuration — Claude's, Cursor's, and plugin sources — and from a
`.mcp.json` in the working directory. In the Nodez repo that exposes servers
pointing at *other vaults*, which is exactly what vault isolation forbids. With
`HOME` pointed at a Nodez-owned directory, a session started in that same repo
reports `mcpToolCount: 0`, and the vault's own server is injected through
`session/new`.

Two traps worth remembering. `grok inspect` still *lists* servers the session
does not load, so it is not a valid isolation check — assert on the session. And
`GROK_HOME` and `GROK_CONFIG_PATH` do **not** work for this; only `HOME` does.

**All three permission modes are expressible.**

| Nodez mode | Grok mechanism |
| --- | --- |
| Read-only | `--sandbox read-only`, kernel-enforced |
| Ask for approval | default mode; ACP delivers the prompt to the client |
| Full access | `--always-approve` |

Read-only is stronger than on the other providers because the operating system
enforces it rather than the agent's own policy. Grok's sandbox blocks
child-process network access on Linux only; on macOS that part is a no-op while
write restriction still applies.

**Approvals work.** ACP carries `session/request_permission`, so the approval
cards built for Codex — unused since Claude could not support them — finally have
a second provider behind them.

## What carries over free

`claude_diff.rs` works unchanged: Grok's `search_replace` takes `file_path`,
`old_string` and `new_string`, the same fields as Claude's `Edit`. The token
arithmetic is identical, since Grok reports the same four usage fields. The
panel, settings, carried context, event contract, command dispatch and shared
session plumbing are all untouched.

Grok is better than Claude in two places: tool results carry a real `exit_code`,
and `session/new` returns the model list with context sizes, so the token meter
gets a true limit and no discovery call is needed.

It is worse in one: ACP declares `promptCapabilities.image: false`, so image
attachments are unavailable and the composer's image control hides for Grok.

## Known risks

The controlled `HOME` is undocumented behaviour, not a supported flag. A Grok
release could add a discovery path that ignores it, silently widening what a
session can reach. The planned guard test — a real session in a directory holding
a `.mcp.json`, asserting zero tools — is the whole mitigation.

Nodez holding a credential is new. The key must not reach the webview, logs,
crash reports, or a conversation transcript.

## Out of scope

Image attachments, concurrent providers, and approvals for Claude, which remain
impossible for reasons recorded in [[Claude Code Provider]].

Related: [[Claude Code Provider]], [[Codex Chat Implementation]],
[[Code Editor and AI Chat Plan]], [[Next Steps]], [[Decision Log]].
