---
title: 面向 Agent 的 MCP
description: 通过 Model Context Protocol 把 VectorFS 暴露给 AI Agent。
weight: 30
---

VectorFS 以两种形态提供 stdio MCP（Model Context Protocol）服务器。根据 Agent
在哪里运行、是否应该持有写者锁来选择。

## `vectorfs mcp` —— 本地写者 {#server-mcp}

以单用户写者的身份，让服务端二进制直接面对数据目录运行：

```bash
vectorfs mcp [--tools ro|rw]
```

它持有每桶写者锁，因此守护进程与本地 Agent 不能同时摄取。默认工具集为读写；
`--tools ro` 则只保留查询。

## `vf mcp` —— 远程代理 {#client-mcp}

运行在 Agent 本机，代理到已在运行的守护进程：

```json
{
  "mcpServers": {
    "vf": { "command": "vf", "args": ["mcp"] }
  }
}
```

锁由守护进程持有，代理自己不持锁——多个 Agent 可以共享一个服务端。它还提供
几个数据面工具没有的动词：`get_source`、`get_source_data`、`status`、
`delete_source` 与 `reindex`。`put_source` 接受 `path` 或内联 `content`，
以及可选的 `waitSeconds`（等到索引完成）。

## 工具集 {#tools}

| 区域 | 工具 |
| --- | --- |
| 数据面（s3vectors） | `list_vector_buckets`、`list_indexes`、`query_vectors`、`get_vectors`、`put_vectors`、`delete_vectors` |
| 文件层 | `search`、`list_sources`、`put_source` |

`vectorfs mcp` 与 `vf mcp` 共享以上两个集合；代理额外提供上段列出的动词。
`--tools ro` 只保留读路径。

## 如何选择 {#choosing}

- **CI 或单机 Agent、用自己的数据目录** —— `vectorfs mcp`。
- **Agent 连共享或远程守护进程** —— `vf mcp`；除非 Agent 需要摄取，否则建议
  `--tools ro`。

> [!TIP]
> 把只读工具与 `store_chunk_text = true` 结合，Agent 就能引用源文本回答问题，
  而不是只给出原始分数。
