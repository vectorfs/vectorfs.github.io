---
title: Build from Source
description: Toolchain, build profiles, and running the conformance suites.
weight: 10
---

## Toolchain {#toolchain}

| Requirement | Version |
| --- | --- |
| Rust | 1.94 or newer |
| C toolchain | a working `cc` for native dependencies |
| `cmake` | for pdfium and other native builds |
| `protoc` | protocol buffer compilation |

## Build {#build}

```bash
git clone https://github.com/vectorfs/vectorfs.git
cd vectorfs
cargo build --profile release-prod --workspace
```

`release-prod` is the production profile — the one the release binaries and Docker images use. Plain `cargo build` works for development.

## Conformance suites {#conformance}

Byte-level AWS compatibility is not aspirational; it is asserted. The `conformance/` directory contains 32-step suites for both SDKs:

- **boto3 (Python)** — 32 steps over the s3vectors face, including error shapes.
- **aws-sdk-go-v2 (Go)** — the same coverage from a second SDK family.

Run them against a freshly built server; they are the fastest signal that a protocol change did not silently break an AWS client. The suites also cover the `/x-vf/` operations through `vf` and the MCP tools.

## Testing your changes {#testing}

- `vectorfs pipelines check` — validates pipeline configuration with exit-code semantics; cheap enough for a pre-commit hook.
- `vectorfs doctor` — probes a live backend's conditional writes, multipart support, model readiness, disk space, and engine drift; useful after storage-layer changes.
- The emulator profile (`--backend memory --no-auth`) gives the suites a volatile server with no cleanup between runs.

## Reporting issues {#issues}

File problems at `github.com/vectorfs/vectorfs/issues`. For protocol-compatibility reports, include the SDK, SDK version, and the operation name — the conformance suites will reproduce most cases directly.
