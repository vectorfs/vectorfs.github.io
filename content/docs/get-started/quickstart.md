---
title: Quickstart
description: Initialize the server, drop in a file, and search it by meaning.
weight: 30
---

This guide walks the five-minute path: configure the client, initialize a server, ingest a file, and query it semantically.

## 1. Configure the client {#configure}

```bash
vf config init --endpoint http://127.0.0.1:8787 \
  --access-key <AK> --secret-key <SK>
```

The configuration lands in `~/.config/vectorfs/config.toml`. Precedence is `--flag` > `VF_*` environment variables > the file — so CI can override with `VF_ENDPOINT` alone.

## 2. Initialize the server {#init}

```bash
vectorfs init --preset text-rag --data-dir ./data
```

`init` writes `vectorfs.toml`, downloads the embedding model with progress and sha256 checks, and generates credentials. The presets differ in what they prepare:

| Preset | Model download | Purpose |
| --- | --- | --- |
| `dev` | none | Protocol-only: SDK emulator, no file layer model |
| `text-rag` | gte-multilingual-base (≈1.2 GiB) | Full text pipeline, local ONNX embedding |
| `air-gapped` | pre-provisioned | Local model, `offline = true`, no outbound calls |

## 3. Serve {#serve}

```bash
vectorfs serve --data-dir ./data
```

The server now exposes all 21 s3vectors operations plus the 8 `/x-vf/` operations, the MCP entry point, the ingest worker, and the reconciler. Check `vf status` (which reads `/health` plus a `GetIngestStatus` summary) or `vectorfs doctor` for a deeper probe.

## 4. Put a file {#put}

```bash
vf put report.pdf --bucket docs
```

The command stays in the foreground until the source reaches `INDEXED`. Prefer fire-and-forget? Add `--async` and poll later with `vf sources status`.

## 5. Search by meaning {#search}

```bash
vf search "capacity planning" --bucket docs
```

Results come back ranked, each carrying `x-vf-src-*` provenance — you always know which file and which model produced a hit. Add `--top-k N` to widen the result set, `--filter JSON` to constrain metadata, `--json` for scripts.

## Inspect and clean up {#inspect}

```bash
vf ls                          # list ingest buckets
vf sources list --bucket docs  # everything ingested, with status
vf get docs/report.pdf         # fetch the stored object back
vf rm docs/report.pdf          # delete it (and its derived vectors, via GC)
```

## Where to go next {#next}

- [File pipeline](/docs/tutorial/file-pipeline/) — tune extraction, chunking, and embedding.
- [S3 Vectors emulator](/docs/tutorial/emulator/) — point boto3 at the server.
- [MCP](/docs/tutorial/mcp/) — give an agent read-only or read-write tools.
