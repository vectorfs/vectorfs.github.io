---
title: Installation
description: Install the client, the server, or both — by script, Docker, or source.
weight: 20
---

## Install script (recommended) {#script}

Install the `vf` client, or the client plus the `vectorfs` server:

```bash
curl -fsSL https://raw.githubusercontent.com/vectorfs/vectorfs/main/scripts/install-cli.sh | bash
```

Options:

| Option | Effect |
| --- | --- |
| `--with-server` | Also install the `vectorfs` server binary |
| `--version v0.1.0` | Install a specific release |
| `--to /usr/local/bin` | Choose the install directory |

Windows (client only):

```powershell
iwr -useb https://raw.githubusercontent.com/vectorfs/vectorfs/main/scripts/install-cli.ps1 | iex
```

Verify:

```bash
vf --version
vectorfs --version   # if you installed the server
```

## Docker {#docker}

The official image is `ghcr.io/vectorfs/vectorfs` — multi-arch, non-root, models kept out of image layers:

```bash
docker compose -f deploy/docker-compose.yml up
```

Set `VF_PRESET=text-rag` on first boot so the container downloads the model into its volume. The compose file ships an `emulator` profile (memory backend, `--no-auth`) for CI. Details in the [deployment guide](/docs/tutorial/deploy/).

## From source {#source}

Building the workspace yourself requires Rust 1.94+, a C toolchain, `cmake`, and `protoc`:

```bash
git clone https://github.com/vectorfs/vectorfs.git
cd vectorfs
cargo build --profile release-prod --workspace
```

## What next {#next}

Configure the client and run your first search in the [Quickstart](/docs/get-started/quickstart/).
