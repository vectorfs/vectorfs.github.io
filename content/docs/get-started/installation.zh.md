---
title: 安装
description: 用脚本、Docker 或源码安装客户端、服务端或两者。
weight: 20
---

## 安装脚本（推荐） {#script}

安装 `vf` 客户端，或客户端加 `vectorfs` 服务端：

```bash
curl -fsSL https://raw.githubusercontent.com/vectorfs/vectorfs/main/scripts/install-cli.sh | bash
```

可选项：

| 选项 | 作用 |
| --- | --- |
| `--with-server` | 同时安装 `vectorfs` 服务端二进制 |
| `--version v0.1.0` | 安装指定版本 |
| `--to /usr/local/bin` | 指定安装目录 |

Windows（仅客户端）：

```powershell
iwr -useb https://raw.githubusercontent.com/vectorfs/vectorfs/main/scripts/install-cli.ps1 | iex
```

验证：

```bash
vf --version
vectorfs --version   # 如果你安装了服务端
```

## Docker {#docker}

官方镜像为 `ghcr.io/vectorfs/vectorfs` —— 多架构、非 root、模型不进镜像层：

```bash
docker compose -f deploy/docker-compose.yml up
```

首次启动设置 `VF_PRESET=text-rag`，让容器把模型下载进卷。compose 文件自带
`emulator` profile（memory 后端、`--no-auth`），适合 CI。详见
[部署指南](/zh/docs/tutorial/deploy/)。

## 从源码构建 {#source}

自行构建整个 workspace 需要 Rust 1.94+、C 工具链、`cmake` 与 `protoc`：

```bash
git clone https://github.com/vectorfs/vectorfs.git
cd vectorfs
cargo build --profile release-prod --workspace
```

## 接下来 {#next}

在[快速开始](/zh/docs/get-started/quickstart/)里配置客户端，跑出第一次搜索。
