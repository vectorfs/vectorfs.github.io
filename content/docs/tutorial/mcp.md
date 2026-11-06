---
title: MCP for Agents
description: Expose VectorFS to AI agents over the Model Context Protocol.
weight: 30
---

VectorFS ships a stdio MCP (Model Context Protocol) server in two forms. Pick based on where your agent runs and whether it should own the writer lock.

## `vectorfs mcp` — local writer {#server-mcp}

Runs the server binary as a single-user writer against a data directory:

```bash
vectorfs mcp [--tools ro|rw]
```

It holds the per-bucket writer lock, so the daemon and a local agent cannot ingest concurrently. Default tool set is read-write; `--tools ro` restricts to queries only.

## `vf mcp` — remote proxy {#client-mcp}

Runs on the agent's machine and proxies to an already-running daemon:

```json
{
  "mcpServers": {
    "vf": { "command": "vf", "args": ["mcp"] }
  }
}
```

Because the daemon owns the lock, the proxy itself does not — multiple agents can share one server. It also exposes a few extra verbs the data-plane tools lack: `get_source`, `get_source_data`, `status`, `delete_source`, and `reindex`. `put_source` accepts a `path` or inline `content`, plus an optional `waitSeconds` to block until indexed.

## Tool sets {#tools}

| Area | Tools |
| --- | --- |
| Data plane (s3vectors) | `list_vector_buckets`, `list_indexes`, `query_vectors`, `get_vectors`, `put_vectors`, `delete_vectors` |
| File layer | `search`, `list_sources`, `put_source` |

`vectorfs mcp` and `vf mcp` share both sets; the proxy adds the extras listed above. With `--tools ro`, only the read paths remain.

## Choosing {#choosing}

- **CI or single workstation agent, own data dir** — `vectorfs mcp`.
- **Agent talking to a shared or remote daemon** — `vf mcp`, ideally `--tools ro` unless the agent must ingest.

> [!TIP]
> Combine read-only tools with `store_chunk_text = true` so agents can answer questions with quotable source text instead of raw scores.
