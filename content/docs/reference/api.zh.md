---
title: API 参考
description: s3vectors 面、原生 /x-vf 面、鉴权与错误。
weight: 30
---

一个 HTTP 服务端，两张 API 面。本页列出两个接口及它们共享的行为。操作级别的
schema 遵循 `s3vectors-2025-07-15` 协议；仓库的 `conformance/` 套件针对 boto3
与 aws-sdk-go-v2 断言了字节级保真。

## s3vectors 面 {#s3vectors}

S3 Vectors 协议的全部 21 个操作，命名与 `s3vectors-2025-07-15` 完全一致：

| 区域 | 操作 |
| --- | --- |
| 向量桶 | `CreateVectorBucket`、`GetVectorBucket`、`ListVectorBuckets`、`DeleteVectorBucket` |
| 桶策略 | `GetVectorBucketPolicy`、`PutVectorBucketPolicy`、`DeleteVectorBucketPolicy` |
| 索引 | `CreateIndex`、`GetIndex`、`ListIndexes`、`DeleteIndex` |
| 向量与查询 | `PutVectors`、`GetVectors`、`DeleteVectors`、`ListVectors`、`QueryVectors` |
| 索引模式 | `PutVectorBucketDefaultIndexMode`、`UpdateIndexMode` |
| 标签 | `TagResource`、`UntagResource`、`ListTagsForResource` |

请求与响应——包括错误形状——与 AWS 逐字节兼容。把任何支持 SigV4 的 SDK 指向
该端点，它的行为就像在与真实服务通信：HTTP 状态码（`400`、`404`、`409`、
`402`、`429`）、`X-Amzn-Errortype` 头、错误 body 中的 `__type`，全都一致。
读取是强一致的读后写。SDK 示例见[模拟器指南](/zh/docs/tutorial/emulator/)。

> [!NOTE]
> 操作级别的请求与响应参数，请以 AWS S3 Vectors API 参考及仓库中的一致性
> 套件为准。

## vectorfs 面 {#vectorfs}

`/x-vf/<Op>` 命名空间下的 8 个原生操作——文件进，语义搜索出：

| 操作 | 用途 |
| --- | --- |
| `PutSource` | 把源对象摄取进文件层 |
| `GetSource` | 获取源的记录与状态 |
| `GetSourceData` | 获取源的字节 |
| `ListSources` | 列出源及其状态 |
| `DeleteSource` | 删除源；GC 回收派生向量 |
| `GetIngestStatus` | 摄取进度摘要 |
| `Search` | 跨桶语义搜索 |
| `Reindex` | 管线变更后触发全量重嵌入 |

异步动词（异步 `PutSource`、`Reindex`）返回 `202`，并通过状态轮询
（`GetIngestStatus`）跟进。`vf` CLI 与 MCP 工具都构建在这张面上。

## 共享行为 {#shared}

- **鉴权** —— 两张面都用 SigV4，每个安装一套凭据（由 `vectorfs init` 生成）。
  `--no-auth` 为本地模拟器接受任意凭据。
- **错误模型** —— 两张面使用同一信封；机器可读的错误码，s3vectors 侧与
  AWS 完全一致。
- **并发** —— 文件层每桶一个写者，由 TTL 心跳 `LOCK` 对象强制。s3vectors
  写入不受该机制串行化。
- **出网** —— s3vectors 面零出站调用；文件层只有模型下载或已配置的嵌入端点
  才出网。

## 限额 {#limits}

结构性限额——批量大小、维度、元数据边界、topK——与 AWS 完全一致并被同样
执行。见[限额参考](/zh/docs/reference/limits/)。
