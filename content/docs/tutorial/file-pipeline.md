---
title: The File Pipeline
description: Tune normalization, extraction, chunking, and embedding for your corpus.
weight: 10
---

Every source object — a file in a source bucket — flows through the same pipeline. This guide explains each stage and the knobs that control it.

```text
normalize → extract → chunk → embed → index
```

## Normalize {#normalize}

The intake step unpacks containers such as gzip so later stages always see plain bytes. There is nothing to configure.

## Extract {#extract}

Extractors pull text out of a source. In v0.2:

| Format | Extractor |
| --- | --- |
| PDF | `pdf-text` via pdfium, or the pure-Rust `pdf-inspector` |
| Markdown | native |
| Plain text | native |

Selection follows an ordered fallback chain and routes by content, so a file stamped `.pdf` but actually gzipped text still extracts correctly. Under `vectorfs.toml`, each MIME type maps through `[extractor.<mime>]`:

```toml
[extractor."application/pdf"]
kind = "pdf-text"

[ingest.docs]
extract = ["pdf-inspector", "pdf-text"]   # ordered fallback chain
```

An `extract` value may be a single instance name, an ordered array like the one above, or `"auto"` — which must occupy the whole value on its own.

## Chunk {#chunk}

The chunker splits extracted text into retrieval units; each chunk becomes one vector. The default is a token-window strategy that is CJK-safe:

```toml
[ingest.docs]
chunk = { kind = "token-window", size = 512, overlap = 64 }
```

Smaller windows sharpen retrieval on dense, sectioned documents; larger ones preserve more context per hit. Keep some overlap so sentences that straddle a boundary remain retrievable.

## Embed {#embed}

Embedders fan out per chunk. Two kinds are supported:

```toml
# Local ONNX runtime (default under text-rag)
[embedder.text]
kind = "ort"
model = "gte-multilingual-base"

# Any OpenAI-compatible endpoint
[embedder.openai]
kind = "openai"
model = "your-model"
endpoint = "https://api.example.com/v1"
dim = 1024                    # declared for openai embedders
```

For air-gapped installs, pre-provision the model and set `path` plus `offline = true`. A bucket's `embedders` list selects which embedders run; `merge` controls how multi-index results combine — `rrf` (default), `weighted`, or `first`.

## Provenance metadata {#provenance}

Every chunk stores `x-vf-src-*` metadata next to its vector:

| Key | Meaning |
| --- | --- |
| `x-vf-src-uri` | Source object URI (≤100 bytes, filterable) |
| `x-vf-src-name` | Source file name |
| `x-vf-model` | Embedding model used |
| `x-vf-modality` | Modality, `text` in v0.2 |
| `x-vf-chunk-text` | The chunk text itself; drop it with `store_chunk_text = false` |

Because the URI is filterable, `vf search --filter '{"x-vf-src-uri": {"eq": "docs/report.pdf"}}'`-style queries constrain results to one file or prefix.

## Fingerprints and reindexing {#reindex}

The server records a configuration fingerprint per pipeline. Change an extractor, chunker, or embedder and the stored fingerprint no longer matches, so `vectorfs serve` refuses to start rather than silently mixing vectors from two configurations. Choose one:

```bash
vectorfs serve --reindex          # skip the gate and reindex at startup
vf reindex --bucket docs          # trigger it on a running daemon (async verb)
```

`vf reindex` only triggers the job — the daemon executes it. Track progress with `vf sources status`.

## Scoping ingestion {#scoping}

`[ingest.<bucket>]` accepts `include` and `exclude` globs. Omit both to ingest the whole bucket; add an optional `index-prefix` to separate derived indexes by pipeline.
