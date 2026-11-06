---
title: 命令参考
description: 全部 vectorfs 与 vf 命令的用途与语法。
weight: 10
---

两个二进制共享同一套 CLI 契约：带 `schema_version` 的 `--json` 输出、形如
`{code, exit, message, hint}` 的结构化错误、stdout 给结果而 stderr 给进度、
无交互、操作幂等、输出确定（平局按 key 决胜）。权威来源是仓库 `design.md`
的附录 E；本页与其保持一致。

## 服务端：`vectorfs` {#server}

通用 flag：`--data-dir`，守护进程的本地主目录。

### `vectorfs init` {#init}

写入 `vectorfs.toml`、预取模型（带进度、sha256 校验与镜像支持），并生成凭据。

```text
vectorfs init [--preset dev|text-rag|air-gapped] [--listen ADDRESS] [--backend URL] --data-dir DIR
```

### `vectorfs serve` {#serve}

提供全部 21 个 s3vectors 操作加 8 个 `/x-vf/` 操作，运行摄取 worker 与对账器，
并暴露 MCP 入口。

```text
vectorfs serve --data-dir DIR [--listen ADDRESS] [--no-auth] [--reindex]
```

`--reindex` 跳过管线指纹门禁，并在启动时重建索引。

### `vectorfs status` {#server-status}

本地总览：锁的持有、后端连通性，以及每桶的 bucket/index/source 计数。远程
形态用 `status --endpoint URL`。

### `vectorfs doctor` {#doctor}

探查后端条件写、multipart 支持、模型就绪、磁盘空间，并统计每桶引擎漂移的
源数量（仅告警）。

### `vectorfs optimize` {#optimize}

Lance 压缩 / IVF_PQ 维护。

```text
vectorfs optimize [--bucket BUCKET] [--index INDEX]
```

### `vectorfs pipelines check` {#pipelines-check}

校验管线配置；可在 CI 中运行，带退出码语义。`vectorfs pipelines list` 打印
已物化的管线、派生索引与指纹。

### `vectorfs mcp` {#server-mcp}

以单用户写者身份运行 stdio MCP 服务器。`[--tools ro|rw]` 选择工具集。见
[面向 Agent 的 MCP](/zh/docs/tutorial/mcp/)。

## 客户端：`vf` {#client}

端点来自 `--endpoint` 或 `VF_ENDPOINT`；凭据与默认值来自
`~/.config/vectorfs/config.toml`。

### `vf put` {#put}

上传文件并等待 `INDEXED`（或异步执行）。

```text
vf put <file…> <bucket>/[<prefix>/] [--async]
vf put <file…> --bucket <b> [--key <k>] [--async]
```

目标位置以 `/` 结尾表示 aws-cp 风格的前缀上传。位置参数目标与 `--bucket`
互斥（用法错误）。

### `vf search` {#search}

在一个桶上做语义搜索。

```text
vf search "<文本>" [<bucket>] [--top-k N] [--filter JSON] [--json]
```

### `vf ls` {#ls}

不带参数时列出摄取桶；带桶（及可选前缀）时列出已摄取的文件，可用
`--status` 过滤。

### `vf sources list` / `vf sources status` {#sources}

```text
vf sources list --bucket <b> [--status S]
vf sources status <key> --bucket <b>
```

### `vf get` / `vf cat` {#get}

```text
vf get <bucket>/<key> [--out FILE]
vf get <key> --bucket <b> [--out FILE]   # 位置参数是完整 key
vf cat <bucket>/<key>                     # 输出到 stdout
```

### `vf rm` {#rm}

删除源对象；GC 会回收其派生向量。

```text
vf rm <bucket>/<key>
vf rm <key> --bucket <b>
```

### `vf reindex` {#reindex}

在管线变更后触发全量重嵌入——实际执行者是守护进程。用 `vf sources status`
轮询进度。

### `vf status` {#client-status}

`/health` 加 `GetIngestStatus` 摘要。

### `vf mcp` {#client-mcp}

针对运行中守护进程的远程代理 MCP 模式。见[面向 Agent 的 MCP](/zh/docs/tutorial/mcp/)。

### `vf config init` {#config-init}

写入 `~/.config/vectorfs/config.toml`。见[配置](/zh/docs/reference/configuration/)。

## 退出码 {#exits}

结构化错误携带 `{code, exit, message, hint}`；每类失败都有对应文档化的退出码，
`--json` 会为脚本包装它们。有意推迟的能力：`cp`/`mv` 统一、`sync`、递归 `rm`
与别名管理。
