---
title: Design Documents
description: Where the authoritative decisions live, from the command table to the v3 bucket model.
weight: 20
---

The VectorFS repository keeps its design history in the open. These documents are the authority when behavior and marketing disagree.

## Document map {#map}

| Document | What it holds |
| --- | --- |
| `design.md` | The master design, including Appendix E — the authoritative command table behind the [command reference](/docs/reference/commands/) |
| `design-v3.md` | The v3 bucket model: source bucket plus `-vf` system bucket, locking, reconciliation |
| `CONTEXT.md` | The terminology glossary — the domain language (`source`, `chunk`, `reindex`, presets, backends) used across code, docs, and issues |
| `docs/adr/0001` | The naming ADR: the `vf` binary name and its collision with fish's VirtualFish, adjudicated |
| `spike/REPORT.md` | Findings from the pre-build spike that shaped the architecture |
| `conformance/` | The dual-SDK, 32-step conformance suites |

## How decisions are recorded {#adr}

Cross-cutting decisions get an ADR (architecture decision record) under `docs/adr/`. ADR 0001 settled the client binary's name — `vf` — accepting the collision with fish's VirtualFish plugin and documenting the reasoning. When you face a naming or compatibility question, check the ADRs first; when you settle one, add a record.

## Terminology discipline {#terminology}

`CONTEXT.md` is the source of truth for words: a **source** is an object in a source bucket (not a "document"); a **chunk** is the retrieval unit with one vector; **reindex** is the explicit full re-embedding after pipeline changes. Code, commit messages, and issue reports all use these terms — please extend the same discipline to your contributions.

## Website sources {#website}

This site lives in the [`vectorfs/vectorfs.github.io`](https://github.com/vectorfs/vectorfs.github.io) repository and builds with [OINK](https://oink.pgsty.com/) on Hugo. See its README for local preview instructions.
