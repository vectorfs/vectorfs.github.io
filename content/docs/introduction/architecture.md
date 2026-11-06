---
title: Architecture
description: Two API faces, the bucket model, backends, the file layer, locking, and garbage collection.
weight: 20
---

VectorFS ships as two binaries — `vectorfs`, the server, and `vf`, the client — over a storage backend that you choose. This page walks through the pieces and how they interact.

## Two API faces {#two-api-faces}

The server exposes one HTTP surface with two faces:

- **s3vectors API** — all 21 operations of the `s3vectors-2025-07-15` protocol. Request and error shapes are AWS byte-compatible, verified by 32-step conformance suites run against unmodified `boto3` and `aws-sdk-go-v2`.
- **vectorfs API** — 8 native operations under `/x-vf/<Op>`: `PutSource`, `GetSource`, `GetSourceData`, `ListSources`, `DeleteSource`, `GetIngestStatus`, `Search`, and `Reindex`.

Both faces authenticate with SigV4 and share one error model. Asynchronous verbs return `202` plus status polling. Reads are strongly consistent — read-after-write — on both faces.

The s3vectors face makes zero outbound network calls. The file layer only reaches the network for model downloads or an explicit embedding endpoint.

## Bucket model {#bucket-model}

Each logical vector bucket `X` maps to a pair of physical buckets:

```text
X/        source bucket  — pure user files, nothing else
X-vf/     system bucket  — LOCK, bucket.json, sources/, indexes/
```

- Anything a standard tool writes into `X/` gets ingested by the file layer.
- The `-vf` suffix is the `bucket_suffix` setting, fixed at `init` time.
- Deleting the vector bucket deletes `X-vf/` but **never** touches `X/`.

## Backends {#backends}

| Backend | Endpoint form | Use |
| --- | --- | --- |
| `file` | local directory (default) | Zero-configuration local runs; the data dir is the bucket root |
| `memory` | in-process | Volatile storage for CI and emulator mode |
| `s3` | `s3+http://host:port` | Any S3-compatible endpoint; verified on SeaweedFS 4.52 and Cloudflare R2 |

There is no VectorFS storage engine: indexes are Lance datasets, ingest state is JSON, and both live in the backend you picked.

## File layer {#file-layer}

Ingestion turns each source object into searchable chunks through a fixed pipeline:

```text
normalize → extract → chunk → embed
```

1. **Normalize** — unpack containers such as gzip.
2. **Extract** — pull text out of the source. PDFs go through pdfium or the pure-Rust `pdf-inspector`; Markdown and plain text have native extractors. An ordered fallback chain picks the extractor per file.
3. **Chunk** — token-window splitting (defaults: 512 tokens, 64 overlap), safe for CJK text.
4. **Embed** — fan out to a local ONNX runtime (`ort`) or any OpenAI-compatible `/embeddings` endpoint.

Each pipeline has a configuration fingerprint. If the fingerprint no longer matches at startup, the server refuses to start unless you pass `--reindex`. See the [file pipeline guide](/docs/tutorial/file-pipeline/) for extractor, chunker, and embedder settings.

Every chunk stores `x-vf-src-*` provenance metadata — source URI, source name, model, modality, and the chunk text itself (disable storage with `store_chunk_text = false`).

## Locking {#locking}

The file layer allows a single writer per bucket. Writers hold a TTL-heartbeat `LOCK` object in `X-vf/`; a crashed writer's lock expires and another can take over. `vectorfs mcp` runs as a single-user writer, while `vf mcp` proxies to a daemon without holding the lock itself.

## Reconciliation and GC {#gc}

A reconciler continuously compares source objects with derived indexes. Garbage collection deletes only vectors whose keys match the derived-chunk format **and** whose source object is gone. Vectors written through the s3vectors face are never touched.

## Where to go next {#next}

- Run it yourself: [Quickstart](/docs/get-started/quickstart/)
- Every command: [Command reference](/docs/reference/commands/)
- Wire up SDKs or agents: [S3 Vectors emulator](/docs/tutorial/emulator/), [MCP](/docs/tutorial/mcp/)
