---
title: 快速开始
description: 初始化服务端，放入一个文件，按语义查询它。
weight: 30
---

本指南走一条五分钟路径：配置客户端、初始化服务端、摄取一个文件、按语义查询。

## 1. 配置客户端 {#configure}

```bash
vf config init --endpoint http://127.0.0.1:8787 \
  --access-key <AK> --secret-key <SK>
```

配置写入 `~/.config/vectorfs/config.toml`。优先级为 `--flag` > `VF_*` 环境变量 >
配置文件 —— 所以 CI 只靠 `VF_ENDPOINT` 就能覆盖。

## 2. 初始化服务端 {#init}

```bash
vectorfs init --preset text-rag --data-dir ./data
```

`init` 会写入 `vectorfs.toml`、带进度与 sha256 校验地下载嵌入模型，并生成凭据。
各预设准备的东西不同：

| 预设 | 模型下载 | 用途 |
| --- | --- | --- |
| `dev` | 无 | 仅协议：SDK 模拟器，不为文件层下载模型 |
| `text-rag` | gte-multilingual-base（约 1.2 GiB） | 完整文本管线，本地 ONNX 嵌入 |
| `air-gapped` | 预置 | 本地模型，`offline = true`，零出网 |

## 3. 启动服务 {#serve}

```bash
vectorfs serve --data-dir ./data
```

服务端此时暴露全部 21 个 s3vectors 操作、8 个 `/x-vf/` 操作、MCP 入口、摄取
worker 与对账器。用 `vf status`（读取 `/health` 加 `GetIngestStatus` 摘要）查看
状态，或用 `vectorfs doctor` 做更深的探查。

## 4. 放入文件 {#put}

```bash
vf put report.pdf --bucket docs
```

该命令在前台等到源对象到达 `INDEXED`。想即发即忘？加 `--async`，稍后用
`vf sources status` 轮询。

## 5. 按语义搜索 {#search}

```bash
vf search "容量规划" --bucket docs
```

结果按相似度排序返回，每条都带 `x-vf-src-*` 溯源信息——命中来自哪个文件、
用的哪个模型，一目了然。加 `--top-k N` 扩大结果集，`--filter JSON` 约束元数据，
`--json` 交给脚本处理。

## 查看与清理 {#inspect}

```bash
vf ls                          # 列出摄取桶
vf sources list --bucket docs  # 已摄取的全部内容与状态
vf get docs/report.pdf         # 取回存储的对象
vf rm docs/report.pdf          # 删除它（GC 会连带清理派生向量）
```

## 接下来 {#next}

- [文件管线](/zh/docs/tutorial/file-pipeline/) —— 调整解析、分块与嵌入。
- [S3 Vectors 模拟器](/zh/docs/tutorial/emulator/) —— 把 boto3 指向服务端。
- [MCP](/zh/docs/tutorial/mcp/) —— 给 Agent 只读或读写工具。
