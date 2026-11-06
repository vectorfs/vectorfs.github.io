---
title: Deployment
description: Run VectorFS under Docker and against S3-compatible storage.
weight: 40
---

## Docker Compose {#docker}

The repository ships `deploy/docker-compose.yml`:

```bash
git clone https://github.com/vectorfs/vectorfs.git
cd vectorfs
docker compose -f deploy/docker-compose.yml up
```

The image `ghcr.io/vectorfs/vectorfs` is multi-arch and runs as a non-root user. Two decisions are made for you:

- **Models stay out of image layers.** Volumes cover `data` and `models`, so an image upgrade does not re-download 1.2 GiB.
- **`VF_PRESET=text-rag` on first boot** primes the model volume before you switch to a leaner environment.

Health checks hit `/health`. The compose file also defines an `emulator` profile — memory backend, `--no-auth` — for CI:

```bash
docker compose -f deploy/docker-compose.yml --profile emulator up
```

Building the image locally? `deploy/docker_build.sh` is the standard path, and `docker_build_cn.sh` routes base-image pulls through mainland-China mirrors.

## The `s3` backend {#s3-backend}

Point the server at any S3-compatible endpoint with the `s3+scheme://host:port` endpoint form:

```bash
vectorfs init --backend s3+http://127.0.0.1:8333 --preset text-rag --data-dir ./data
vectorfs serve --data-dir ./data
```

Verified combinations include SeaweedFS 4.52 and Cloudflare R2. The bucket model does not change: your source bucket stays plain objects, and the `-vf` system bucket holds locks, state, and Lance indexes — all in your storage.

Before committing to a backend, run `vectorfs doctor`. It probes conditional writes, multipart support, model readiness, disk space, and per-bucket engine drift, so an unsupported store fails loudly at day zero instead of corrupting quietly in week three.

## Environment overrides {#env}

In Docker, `VF_*` variables override the matching scalar settings — endpoint, credentials, preset, backend. Complex structures (extractor chains, chunkers, embedder blocks) belong in `vectorfs.toml`; mount or bake that file instead of flattening it into env vars. Precedence is unchanged: flag > `VF_*` env > file > default.

## Operational checklist {#checklist}

- One writer per bucket. Run exactly one ingest daemon per bucket, or let `vectorfs mcp` own it — TTL heartbeats will resolve crashes, not contention.
- Watch `vf status` in your monitor of choice; it is a cheap `/health` plus `GetIngestStatus` call.
- Change a pipeline setting? Expect the fingerprint gate; plan the `--reindex` window. See [the file pipeline](/docs/tutorial/file-pipeline/#reindex).
- Deleting a vector bucket removes only the `-vf` side. Your files in the source bucket are never touched.
