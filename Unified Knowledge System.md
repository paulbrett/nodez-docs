---
id: diamante-unified-knowledge-system
title: Unified Knowledge System
type: workflow
status: active
created: 2026-08-19
updated: 2026-08-21
tags:
  - obsidian
  - graphify
  - workflow
  - dakila
---

# Unified Knowledge System

Diamante should become a unified system that combines the human-first vault workflow of Obsidian with the relationship-first graph workflow of Graphify.

The Dakila workflow is the reference model.

## Product Thesis

Diamante is not only a Markdown editor and not only a graph visualizer.

It is a local-first workspace where notes, code, decisions, issues, artifacts, and dependency relationships share one navigable knowledge layer.

## Source-Of-Truth Split

Diamante should support three complementary truth layers:

- **GitHub/source folders** are implementation truth. Code, tests, package manifests, firmware, app screens, and repo docs describe what currently exists.
- **Obsidian-style vault notes** are human-readable intent. Architecture, decisions, roadmap, experiments, meeting notes, handoffs, and operator context live as Markdown.
- **Graph layer** is relationship truth. It connects notes, code symbols, files, components, decisions, versions, features, and dependencies into a queryable graph.

These layers must not compete.

## Conflict Resolution

When sources disagree:

1. For current implementation behavior, source code and checked artifacts win.
2. For rationale, design intent, and roadmap direction, the latest accepted vault decision wins.
3. For relationships and dependency impact, the graph layer explains connections but must cite whether each edge is extracted or inferred.
4. When the disagreement affects architecture, safety, security, sync, or data ownership, Diamante should surface the conflict instead of silently resolving it.

## Obsidian-Inspired Capabilities

Diamante should preserve the parts of Obsidian that make a vault feel durable and personal:

- local folders of Markdown files
- YAML frontmatter properties
- wikilinks and aliases
- backlinks and unlinked mentions
- local graph and global graph
- graph filters for tags, groups, attachments, existing files, and orphaned notes
- command palette
- fast search
- Git-backed sync without proprietary cloud dependency

## Graphify-Inspired Capabilities

Diamante should add the parts of Graphify that make a project graph useful for engineering work:

- AST-based code extraction where possible
- nodes for files, folders, symbols, packages, docs, decisions, features, and components
- edges for imports, calls, definitions, references, dependencies, decisions, and wikilinks
- edge provenance: `extracted`, `inferred`, or `manual`
- communities and clusters for subsystems
- most-connected nodes for system hubs
- query, path, and explain views
- project-scoped graph output that agents can query before reading many files

## Dakila-Style Workflow

For a project like Dakila, Diamante should support this loop:

1. Open the docs vault and source repo as one workspace.
2. Pull latest changes from GitHub.
3. Build or update the graph from Markdown, YAML, code, and package files.
4. Ask relationship questions before editing.
5. Read the specific source files and notes that the graph points to.
6. Make the implementation change.
7. Update the relevant vault note or decision record.
8. Rebuild the graph.
9. Commit code, docs, and graph updates together when appropriate.

## Graph Model

Minimum node types:

- note
- file
- folder
- symbol
- package
- decision
- feature
- component
- tag
- artifact

Minimum edge types:

- links_to
- backlinks_to
- imports
- calls
- defines
- depends_on
- references
- documents
- decides
- supersedes
- related_to

Every edge should store:

- source node
- target node
- edge type
- provenance
- confidence
- source file/path
- optional line or heading reference

## Interface Model

The current graph modal is the seed.

Next it should grow into:

- preview graph above Backlinks
- full graph modal
- local graph mode for the active note
- global graph mode for the whole workspace
- search box
- filters by node type, edge type, tag, folder, and provenance
- hover/click explanation panel
- path finder between two nodes
- graph query panel

## Agent Model

Agents should use Diamante in this order:

1. Read `AGENTS.md`.
2. Query the graph when the task depends on relationships, dependencies, architecture, features, symbols, or impact.
3. Read the relevant vault notes for intent and decisions.
4. Read source files for implementation truth.
5. Update code, docs, and graph outputs when a meaningful change lands.

Setup and MCP completeness track: [[Agent and Human Setup]].

Related: [[Architecture]], [[Backlinks and Indexing]], [[GitHub Sync]], [[Product Roadmap]], [[Decision Log]], [[Agent and Human Setup]], [[Next Steps]]
