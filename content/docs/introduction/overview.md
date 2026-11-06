---
title: Project Overview
description: What VectorFS does, who it serves, and where it is heading.
weight: 10
---

VectorFS is a self-hosted, single-binary system that combines an **S3 Vectors-compatible vector store** with **file vectorization**: you drop files in, and semantic search comes out. In local or memory mode it doubles as a development and CI emulator for the S3 Vectors protocol.

## The problem {#problem}

Semantic search usually forces a choice. Either you run a separate vector database and write code to keep it in sync with your files, or you bolt an indexing service onto your storage. Either way there is a second system to deploy, monitor, back up, and explain.

VectorFS removes that second system. It has no storage engine of its own — your data stays as Lance and JSON in the backend you already run.

## The outcome {#outcome}

After adopting VectorFS you can:

- Point unmodified AWS SDKs (`boto3`, `aws-sdk-go-v2`) at a local endpoint and use the full s3vectors protocol.
- Put a PDF, Markdown, or plain-text file into a bucket and query it by meaning, with no pipeline code of your own.
- Trace every result back to its source file through `x-vf-src-*` provenance metadata.
- Do all of the above on one machine, with one server binary and one client binary.

## Two API faces {#two-api-faces}

- The **s3vectors API** implements all 21 operations of `s3vectors-2025-07-15` with AWS-identical, byte-compatible errors.
- The **vectorfs API** adds 8 native operations under `/x-vf/<Op>`: files in, semantic search out.

Both faces use SigV4 authentication and share one error model. See [Architecture](/docs/introduction/architecture/) for how they fit together.

## Status and roadmap {#status}

> [!NOTE]
> v0.2 is a pre-release. The text modality is the current MVP: PDF, Markdown, and plain text.

| Milestone | Scope |
| --- | --- |
| v0.2 (current) | Text modality MVP, dual SDK conformance suites, presets, Docker, MCP |
| v0.3 (planned) | Image modality, OCR, docx and HTML extraction, hybrid full-text retrieval |

## Non-goals {#non-goals}

- QPS limits and vector-count quotas are not enforced; only the structural AWS limits are, identically.
- VectorFS does not re-implement storage. Durability, replication, and scaling belong to the backend you choose (`file`, `memory`, or `s3`).
- Full multi-modality beyond text waits for v0.3.

## License {#license}

The VectorFS server and client are Apache-2.0. This website is MIT.
