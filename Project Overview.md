---
id: nodez-project-overview
title: Project Overview
type: overview
status: active
created: 2026-08-19
updated: 2026-08-19
tags:
  - product
  - mvp
---

# Project Overview

Nodez Notes is an Obsidian-like app for personal knowledge bases and Graphify-like project relationship graphs.

It should feel fast, calm, and local-first. Notes should remain useful even if the app disappears, because the vault is just folders and Markdown files.

## Product thesis

Most note apps make sync and storage feel magical. Nodez should make them understandable.

The user owns a vault folder and source folders. GitHub can sync them. The app provides a polished editor, index, graph, and navigation layer on top.

## Unified Direction

Nodez should combine:

- Obsidian's durable local Markdown vault, backlinks, YAML properties, and graph exploration
- Graphify's code/docs relationship graph, edge provenance, communities, path tracing, and query/explain workflow
- Dakila's source-of-truth model: GitHub for implementation truth, Obsidian-style notes for intent, graph layer for relationships

## Non-goals for version 1

- proprietary sync service
- hosted note database
- collaborative editing
- Obsidian plugin compatibility
- mobile app
- public publishing

## Version 1 promise

Open a folder of Markdown files. Edit notes. Link notes with `[[wikilinks]]`. Search fast. See backlinks. Preview the graph. Sync with GitHub.

Related: [[Architecture]], [[Unified Knowledge System]], [[Product Roadmap]], [[GitHub Sync]]
