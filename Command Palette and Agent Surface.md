---
id: diamante-command-palette-agent-surface
title: Command Palette and Agent Surface
type: architecture
status: active
created: 2026-08-20
updated: 2026-08-22
tags:
  - mcp
  - agents
  - ui
  - spec
---

# Command Palette and Agent Surface

Implementation-ready design for the feature after [[Repo Indexing Phase 6a]]: a command palette in the app, a matching read+write surface for AI agents over MCP, and an `AGENTS.md` contract documenting both — so the merged vault+repo graph is something an agent can query *and* act on, not just read.

**Status (2026-08-21):** Palette + graph reads + vault write/read tools + graph freshness + setup wizard / MCP export + **app-repo `AGENTS.md` agent contract** shipped. Dual Hermes MCP servers wired. Follow-on: higher-order graph tools (P4) — [[Agent and Human Setup]].

## Goal

One searchable command palette (`Cmd+K`) covering every existing app action. A parallel MCP tool surface with the same semantics: query tools (already exist) plus new write tools scoped to vault notes only. An `AGENTS.md` in the app repo that documents this contract for both human contributors and AI agents, structured after the reference pattern in `AGENTS-GROK.md` (one root contract + hard rules + pointers to topic docs) — none of that file's actual tech stack applies here.

## Terminology (resolves an ambiguity worth stating explicitly)

"Vault" means two things in this codebase and the design below always says which:

- **The docs vault** — Diamante's own project docs at `~/Documents/Projects/Diamante`. This is `diamante-mcp.mjs`'s current default `vaultDir`, used by AI agents (like Claude) working *on* Diamante itself, per the existing `AGENTS.md`.
- **A configured vault** — whatever folder `DIAMANTE_VAULT_DIR` points at. The MCP server has always been generic over this (the docs vault is just its default); an end user's own notes vault, opened in the Diamante app, is exactly the same kind of thing. When that vault has had a repo indexed alongside it in the app, it also contains `.diamante/graph.json` (from [[Repo Indexing Phase 6a]]).

Everything below operates on "the configured vault," whichever one that is.

## Non-goals

- No MCP write access to an indexed repo, ever — Phase 6a's read-only boundary holds. Repo = implementation truth; agents read it, they don't mutate it through this surface.
- No `cmdk` or other new UI dependency — the palette is hand-rolled, matching this project's established minimal-dependency stance.
- No change to the app UI's own delete flow (native `confirm()`, hard delete) — that already has a human in the loop. Only the MCP path gets the trash-based soft delete, because it doesn't.
- No cross-vault MCP operations — one configured vault per server instance, same as today.

## Frontend: command registry + palette

### `src/commands.ts` (new)

```ts
type Command = {
  id: string;
  label: string;
  hint?: string; // e.g. current theme name, vault path
  run: (ctx: CommandContext) => void;
};
```

`CommandContext` is a plain object of the state setters/handlers `App.tsx` already has (`addNote`, `openVault`, `openSourceRoot`, `startRename`, `deleteActiveNote`, `setGraphOpen`, `setTheme`, `setGraphOrigin`, `loadDummyGraph`, `runSyncPreview`) — the registry doesn't reimplement any behavior, it just names and lists what already exists as a flat, searchable array. Themes and any other enumerable options expand into one command each (e.g. "Switch theme: Forest").

### Palette UI

A new `src/CommandPalette.tsx`, opened by `Cmd+K` (and closable by `Escape`), following the exact `.modalLayer` overlay pattern the Settings and Graph modals already use: a search input, a filtered/keyboard-navigable list (arrow keys + Enter), fuzzy-matched on `label` client-side (simple substring/subsequence match — no new dependency). Selecting a command closes the palette and calls its `run(ctx)`.

## MCP server updates (`scripts/diamante-mcp.mjs`)

### Graph source

`buildGraph()` gets a check before its existing vault-walk logic: if `<vaultDir>/.diamante/graph.json` exists, parse and return it directly (it's already the same node/edge shape). Otherwise, fall back to today's behavior (rebuild from the vault's Markdown alone). This is what makes an indexed repo visible to MCP queries — no change to the 6 existing read tools' behavior or signatures, just a richer graph underneath them when a repo has been indexed into the configured vault.

### New write tools (vault notes only)

- `create_note(title: string): RawNote` — mirrors `create_note` in `lib.rs`: `Untitled` if blank, `{title}.md` at vault root, `# {title}\n\n` starter content, errors if the file already exists.
- `write_note(path: string, content: string): { modifiedMs: number }` — mirrors `write_note`: overwrite content at an existing vault-relative path.
- `rename_note(path: string, newTitle: string): RawNote` — mirrors `rename_note` (rename the file, keep its folder) **and** rewrites `[[wikilinks]]` in every other note in the vault that referenced the old title, using the same regex `extractWikilinks`/rename logic already in `src/noteUtils.ts` (`renameWikilinks`), ported to plain JS in the MCP script — the app's Rust side doesn't do this rewrite itself either; it's `App.tsx` that does it in JS today, so this is genuinely new logic for the MCP script, not a straight port from Rust.
- `delete_note(path: string): { trashedTo: string }` — moves the file to `<vault>/.diamante/trash/<original-relative-path>` (creating parent directories as needed; if a file already exists at that trash destination, suffix with a timestamp rather than overwrite or error). Never a hard delete on this path.

All four use the same `safe_join`-style path-containment check the Rust side already has (guard against `path` escaping the vault root) — the MCP script gets its own small equivalent, since it's a separate Node process with no access to the Rust helper.

## `AGENTS.md` restructure (app repo)

Replace the current flat instruction list with a structure mined from `AGENTS-GROK.md`'s pattern (not its content):

1. **What this is** — one paragraph: Diamante, the source-of-truth split (repo = implementation truth, vault = intent, graph = relationships), same as [[Unified Knowledge System]] already establishes.
2. **Hard rules** (the "never do X" table that pattern uses) — e.g. never write to an indexed source root; `delete_note` via MCP is always soft; one configured vault per server instance.
3. **MCP tool reference** — a table of all 10 tools (6 existing query tools + 4 new write tools), one line each: name, what it does, read or write.
4. **Agent workflow** — restates the existing `AGENTS.md`/[[Unified Knowledge System]] loop (query graph → read vault notes → read source → act), now explicit that "act" can mean calling a write tool, not just editing files directly.

This stays one file (not a `skills/` directory of many small files) — the tool surface is small enough that one reference table is clearer than a folder of one-tool-each playbooks; revisit if the tool count grows a lot. Full layering (MCP vs host skills vs vault knowledge): [[Agent Skills and Surfaces]].

## Data flow

Palette: `Cmd+K` → filter commands by typed text → arrow/Enter selects → `run(ctx)` calls the existing handler exactly as its sidebar/topbar button would. No new state model beyond "is the palette open" and the search text.

MCP: agent calls a tool → script resolves the configured vault (`DIAMANTE_VAULT_DIR` or default) → for reads, serves `.diamante/graph.json` if present else rebuilds from Markdown → for writes, performs the filesystem operation directly (no confirmation prompt — the soft-delete trash is the safety net, not a prompt, since MCP tool calls aren't interactive) → returns the same shape the app's own Rust commands return where applicable (`RawNote`), so a caller already familiar with one surface recognizes the other.

## Error handling

- Palette: a command whose precondition isn't met (e.g. "Open repo" when already mid-pick) is simply not shown as disabled-but-visible cruft — commands that don't make sense in the current state are filtered out of the list entirely, not rendered greyed-out.
- MCP write tools: same error-string convention the existing 6 tools already use (thrown/returned error surfaces as the tool's error result) — `rename_note`/`delete_note` on a path that doesn't exist, `create_note` on a title that collides, path-containment violations, all just return a clear error string, no partial-write states to reason about since each operation is a single `fs` call (or, for rename's wikilink rewrite, a set of independent per-file writes — if one of those fails mid-loop, the note itself has still been renamed; this is the same best-effort behavior `App.tsx`'s own rename already has today, not a new gap introduced here).

## Testing / verification

- No test framework in this project (established); verify via `npm run lint`, `npm run build`, and a manual MCP smoke pass — call each new tool once against a scratch vault (not the real docs vault) and confirm the file-level result on disk, the same way `graph_stats`/`shortest_path` were smoke-tested when the MCP server first shipped.
- Palette: hands-on check in the browser/app (open with `Cmd+K`, search, select a few commands, confirm they do what their existing button does) — same category of verification as this session's other UI work.

Related: [[Repo Indexing Phase 6a]], [[Repo Indexing]], [[Unified Knowledge System]], [[Graphify Tech Research]], [[Architecture]], [[Agent and Human Setup]], [[Next Steps]]
