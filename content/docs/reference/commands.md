---
title: Command Reference
description: Every vectorfs and vf command with purpose and syntax.
weight: 10
---

Two binaries share one CLI contract: `--json` output with a `schema_version`, structured errors of the form `{code, exit, message, hint}`, stdout for results and stderr for progress, no interactivity, idempotent operations, and deterministic output with ties broken by key. The authoritative source is Appendix E of `design.md` in the repository; this page mirrors it.

## Server: `vectorfs` {#server}

Common flag: `--data-dir`, the daemon's local home.

### `vectorfs init` {#init}

Write `vectorfs.toml`, prefetch models (with progress, sha256 checks, and mirror support), and generate credentials.

```text
vectorfs init [--preset dev|text-rag|air-gapped] [--listen ADDRESS] [--backend URL] --data-dir DIR
```

### `vectorfs serve` {#serve}

Serve all 21 s3vectors operations plus the 8 `/x-vf/` operations, run the ingest worker, the reconciler, and expose the MCP entry point.

```text
vectorfs serve --data-dir DIR [--listen ADDRESS] [--no-auth] [--reindex]
```

`--reindex` skips the pipeline fingerprint gate and reindexes at startup.

### `vectorfs status` {#server-status}

Local overview: lock holding, backend connectivity, and per-bucket bucket/index/source counts. Remote form via `status --endpoint URL`.

### `vectorfs doctor` {#doctor}

Probes backend conditional writes, multipart support, model readiness, disk space, and counts engine-drift sources per bucket (warnings only).

### `vectorfs optimize` {#optimize}

Lance compaction / IVF_PQ maintenance.

```text
vectorfs optimize [--bucket BUCKET] [--index INDEX]
```

### `vectorfs pipelines check` {#pipelines-check}

Validate pipeline configuration; CI-runnable with exit-code semantics. `vectorfs pipelines list` prints materialized pipelines, derived indexes, and fingerprints.

### `vectorfs mcp` {#server-mcp}

Run the stdio MCP server as a single-user writer. `[--tools ro|rw]` selects the tool set. See [MCP for agents](/docs/tutorial/mcp/).

## Client: `vf` {#client}

Endpoint from `--endpoint` or `VF_ENDPOINT`; credentials and defaults from `~/.config/vectorfs/config.toml`.

### `vf put` {#put}

Upload a file and wait for `INDEXED` (or run async).

```text
vf put <file…> <bucket>/[<prefix>/] [--async]
vf put <file…> --bucket <b> [--key <k>] [--async]
```

A trailing `/` on the target means aws-cp-style prefix upload. The positional target and `--bucket` are mutually exclusive (usage error).

### `vf search` {#search}

Semantic search over a bucket.

```text
vf search "<text>" [<bucket>] [--top-k N] [--filter JSON] [--json]
```

### `vf ls` {#ls}

With no argument, list ingest buckets; with a bucket (and optional prefix), list ingested files, optionally filtered by `--status`.

### `vf sources list` / `vf sources status` {#sources}

```text
vf sources list --bucket <b> [--status S]
vf sources status <key> --bucket <b>
```

### `vf get` / `vf cat` {#get}

```text
vf get <bucket>/<key> [--out FILE]
vf get <key> --bucket <b> [--out FILE]   # positional is the full key
vf cat <bucket>/<key>                     # stdout form
```

### `vf rm` {#rm}

Delete a source object; GC reclaims its derived vectors.

```text
vf rm <bucket>/<key>
vf rm <key> --bucket <b>
```

### `vf reindex` {#reindex}

Trigger a full re-embedding after pipeline changes — the daemon executes it. Poll with `vf sources status`.

### `vf status` {#client-status}

`/health` plus a `GetIngestStatus` summary.

### `vf mcp` {#client-mcp}

Remote-proxy MCP mode against a running daemon. See [MCP for agents](/docs/tutorial/mcp/).

### `vf config init` {#config-init}

Write `~/.config/vectorfs/config.toml`. See [Configuration](/docs/reference/configuration/).

## Exit codes {#exits}

Structured errors carry `{code, exit, message, hint}`; the documented exit codes accompany each failure class, and `--json` wraps them for scripts. Deferred by design: `cp`/`mv` unification, `sync`, recursive `rm`, and alias management.
