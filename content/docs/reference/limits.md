---
title: Limits
description: Structural limits enforced identically to AWS, and the limits intentionally not enforced.
weight: 40
---

VectorFS enforces the structural limits of the S3 Vectors protocol exactly as AWS does — same bounds, same rejection behavior. QPS throttling and vector-count quotas are deliberately **not** enforced.

## Enforced structural limits {#structural}

| Operation | Limit |
| --- | --- |
| `PutVectors` | ≤ 500 vectors per call, ≤ 20 MiB per call |
| `GetVectors` | ≤ 100 keys per call |
| `DeleteVectors` | ≤ 500 keys per call |
| Vector dimension | 1–4096 |
| Vector metadata | ≤ 50 keys, ≤ 40 KB per vector |
| `QueryVectors` topK | ≤ 10,000 (paginated) |
| `ListVectors` page size | 1–1000, default 500 |
| Query segments | 1–16 |

Distance-metric validation mirrors AWS too: `cosine` rejects zero vectors, and any vector containing `NaN` or `Inf` is rejected.

## Not enforced {#not-enforced}

- **QPS / throttling** — no rate limits are applied.
- **Vector-count quotas** — total vectors per bucket or index are unbounded.

Treat these freedoms as test-time conveniences, not production guarantees. Anything that must behave identically under real S3 Vectors throttling still needs validation against the actual service.

## File-layer limits {#file-layer}

Two file-layer settings bound ingestion rather than API calls: `max_file_size` (default 200 MB) rejects oversized sources, and the scan interval (default 30 s) governs how quickly reconciler sweeps notice bucket changes made by standard tools.
