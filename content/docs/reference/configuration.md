---
title: Configuration
description: config.toml, vectorfs.toml, environment variables, and precedence.
weight: 20
---

VectorFS keeps two files straight: the **client** config at `~/.config/vectorfs/config.toml`, written by `vf config init`, and the **server** config `vectorfs.toml`, written by `vectorfs init` into the data directory.

## Precedence {#precedence}

One chain decides every setting:

```text
--flag  >  VF_* environment variable  >  file  >  built-in default
```

In Docker, `VF_*` variables map onto scalar items — endpoint, credentials, preset, backend. Everything structural lives in TOML.

## Client: `config.toml` {#client-config}

```bash
vf config init --endpoint http://127.0.0.1:8787 \
  --access-key <AK> --secret-key <SK>
```

The file stores the endpoint and SigV4 credentials. `VF_ENDPOINT` (or `--endpoint`) overrides it per invocation — the usual trick for CI.

## Server: `vectorfs.toml` {#server-config}

### `bucket_suffix` {#bucket-suffix}

```toml
bucket_suffix = "-vf"     # default; install-time constant
```

The suffix separating system buckets from source buckets. Fixed at `init`; changing it later orphans existing system buckets.

### `[embedder.<name>]` {#embedder}

```toml
[embedder.text]
kind = "ort"                       # "ort" (local ONNX) or "openai"
model = "gte-multilingual-base"    # default under the text-rag preset
# path = "/models/gte.onnx"        # pre-provisioned model file
# offline = true                   # air-gapped: never fetch
# endpoint = "https://…/v1"        # openai-compatible embedders
# dim = 1024                       # declared dimension for openai
```

### `[extractor.<mime>]` {#extractor}

```toml
[extractor."application/pdf"]
kind = "pdf-text"
```

Per-MIME extractor instances. Pipelines reference them by name; see `extract` under `[ingest.<bucket>]`.

### `[ingest.<bucket>]` {#ingest}

```toml
[ingest.docs]
extract = ["pdf-inspector", "pdf-text"]     # name | ordered array | "auto"
chunk = { kind = "token-window", size = 512, overlap = 64 }
embedders = ["text"]
merge = "rrf"                               # rrf | weighted | first
# index-prefix = "v2"                       # optional
# include = ["**/*.pdf"]                    # optional globs; omit = whole bucket
# exclude = ["**/*.tmp"]
```

`"auto"` must occupy the `extract` value on its own. `store_chunk_text` (default on) controls whether chunk text is kept as metadata.

### Non-TOML runtime settings {#runtime}

Set as flags rather than keys: `--no-auth`, `--backend` (`file`, `memory`, `s3+http(s)://host:port`), `max_file_size` (default 200 MB), scan interval (default 30 s), retries (default 3).

## Environment variables {#env}

| Variable | Effect |
| --- | --- |
| `VF_ENDPOINT` | Client endpoint; overrides `config.toml` |
| `VF_*` (general) | Override the matching scalar setting |
| `HF_ENDPOINT` | Mirror for model downloads (sha256-verified) |
| `AWS_ENDPOINT_URL` | Honored by AWS SDKs pointing at VectorFS |

## Validating {#validating}

`vectorfs pipelines check [--config FILE]` validates pipeline configuration with exit-code semantics — wire it into CI. `vectorfs doctor` additionally probes the live backend and model readiness.
