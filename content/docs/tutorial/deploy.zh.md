---
title: 部署
description: 用 Docker 运行 VectorFS，或对接 S3 兼容存储。
weight: 40
---

## Docker Compose {#docker}

仓库自带 `deploy/docker-compose.yml`：

```bash
git clone https://github.com/vectorfs/vectorfs.git
cd vectorfs
docker compose -f deploy/docker-compose.yml up
```

镜像 `ghcr.io/vectorfs/vectorfs` 是多架构的，并以非 root 用户运行。有两件事
已经替你决定好：

- **模型不进镜像层。** `data` 与 `models` 各有卷，升级镜像不会重新下载
  1.2 GiB。
- **首次启动设置 `VF_PRESET=text-rag`**，先把模型卷填好，再切到更精简的环境。

健康检查走 `/health`。compose 文件还定义了 `emulator` profile —— memory 后端、
`--no-auth` —— 适合 CI：

```bash
docker compose -f deploy/docker-compose.yml --profile emulator up
```

要在本地构建镜像？`deploy/docker_build.sh` 是标准路径；`docker_build_cn.sh`
会让基础镜像的拉取走大陆镜像。

## `s3` 后端 {#s3-backend}

用 `s3+scheme://host:port` 形式的端点，把服务端指向任意 S3 兼容存储：

```bash
vectorfs init --backend s3+http://127.0.0.1:8333 --preset text-rag --data-dir ./data
vectorfs serve --data-dir ./data
```

已验证的组合包括 SeaweedFS 4.52 与 Cloudflare R2。桶模型不变：源桶仍然是
普通对象，`-vf` 系统桶承载锁、状态与 Lance 索引——都在你的存储里。

确定后端之前，先跑 `vectorfs doctor`。它会探查条件写、multipart 支持、模型
就绪、磁盘空间以及每桶的引擎漂移，让不被支持的存储在第零天就大声失败，而
不是在第三周悄悄损坏数据。

## 环境变量覆盖 {#env}

在 Docker 里，`VF_*` 变量覆盖对应的标量设置——端点、凭据、预设、后端。复杂
结构（解析链、分块器、嵌入器块）请放进 `vectorfs.toml`；挂载或烧录该文件，
而不是把它们压平成环境变量。优先级不变：flag > `VF_*` 环境变量 > 文件 > 默认。

## 运维清单 {#checklist}

- 每桶一个写者。每个桶只跑一个摄取守护进程，或让 `vectorfs mcp` 独占——
  TTL 心跳解决的是崩溃，不是争抢。
- 把 `vf status` 接进你顺手的监控；它只是一次廉价的 `/health` 加
  `GetIngestStatus` 调用。
- 改了管线配置？指纹门禁会拦住启动，请为 `--reindex` 预留窗口。见
  [文件管线](/zh/docs/tutorial/file-pipeline/#reindex)。
- 删除向量桶只会删除 `-vf` 一侧。源桶里的文件永不被触碰。
