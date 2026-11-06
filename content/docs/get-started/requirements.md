---
title: Requirements
description: Hardware, operating system, and network prerequisites.
weight: 10
---

## Operating system {#os}

- **Server (`vectorfs`)** — Linux. The Docker image is multi-arch and runs non-root.
- **Client (`vf`)** — Linux and Windows. The Windows installer covers the client only.
- **Docker** — any host that runs Docker works; see the [deployment guide](/docs/tutorial/deploy/).

## Hardware {#hardware}

- **CPU only** — the default `text-rag` preset embeds with a local ONNX runtime; no GPU is required.
- **Memory** — plan for the embedding model plus your workload. The default model, `gte-multilingual-base`, needs roughly 1.2 GiB.
- **Disk** — the model download (≈1.2 GiB) plus your data directory. Indexes grow with the number of chunks, not the number of files.

## Network {#network}

The first `vectorfs init --preset text-rag` downloads the embedding model. If direct access is slow or blocked, point `HF_ENDPOINT` at a mirror. The download is sha256-verified.

After that, a `file`-backend server makes no outbound calls. An `s3` backend talks only to your S3-compatible endpoint, and an OpenAI-compatible embedder talks only to the endpoint you configure.

## Building from source {#from-source}

Building the server yourself needs Rust 1.94 or newer, a C toolchain, `cmake`, and `protoc`. See [Build from source](/docs/develop/build-from-source/).

> [!TIP]
> Just want to try the s3vectors protocol with an existing boto3 test suite? The `dev` preset needs no model download at all.
