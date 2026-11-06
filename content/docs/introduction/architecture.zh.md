---
title: 架构
description: 两张 API 面、桶模型、后端、文件层、锁与垃圾回收。
weight: 20
---

VectorFS 以两个二进制交付——服务端 `vectorfs` 和客户端 `vf`——跑在你选择的
存储后端上。本页过一遍各个部件以及它们如何协作。

## 两张 API 面 {#two-api-faces}

服务端暴露一个 HTTP 入口，上面有两张 API 面：

- **s3vectors API** —— `s3vectors-2025-07-15` 协议的全部 21 个操作。请求与
  错误形状和 AWS 逐字节兼容，由针对未修改 `boto3` 与 `aws-sdk-go-v2` 的
  32 步一致性套件验证。
- **vectorfs API** —— `/x-vf/<Op>` 下的 8 个原生操作：`PutSource`、
  `GetSource`、`GetSourceData`、`ListSources`、`DeleteSource`、
  `GetIngestStatus`、`Search` 与 `Reindex`。

两张 API 面都用 SigV4 鉴权，共享同一套错误模型。异步动词返回 `202` 并通过
状态轮询跟进。两张面上的读取都是强一致的——读后写一致。

s3vectors 面零出网。文件层只有在下载模型或访问显式配置的嵌入端点时才出网。

## 桶模型 {#bucket-model}

每个逻辑向量桶 `X` 对应一对物理桶：

```text
X/        源桶    —— 纯用户文件，别无他物
X-vf/     系统桶  —— LOCK、bucket.json、sources/、indexes/
```

- 标准工具写入 `X/` 的任何内容都会被文件层摄取。
- `-vf` 后缀由 `bucket_suffix` 设置决定，在 `init` 时固定。
- 删除向量桶会删除 `X-vf/`，但**绝不**触碰 `X/`。

## 后端 {#backends}

| 后端 | 端点形式 | 用途 |
| --- | --- | --- |
| `file` | 本地目录（默认） | 零配置本地运行；数据目录即桶根 |
| `memory` | 进程内 | CI 与模拟器模式的易失存储 |
| `s3` | `s3+http://host:port` | 任意 S3 兼容端点；已在 SeaweedFS 4.52 与 Cloudflare R2 上验证 |

VectorFS 没有自己的存储引擎：索引是 Lance 数据集，摄取状态是 JSON，二者都
留在你选择的后端里。

## 文件层 {#file-layer}

摄取把每个源对象变成可检索的分块，经过固定的管线：

```text
normalize → extract → chunk → embed
```

1. **normalize（规范化）** —— 解开 gzip 等容器。
2. **extract（解析）** —— 从源文件抽出文本。PDF 走 pdfium 或纯 Rust 的
   `pdf-inspector`；Markdown 与纯文本有原生解析器。按有序回退链为每个文件
   选择解析器。
3. **chunk（分块）** —— token 窗口切分（默认：512 token、重叠 64），对 CJK
   文本安全。
4. **embed（嵌入）** —— 扇出到本地 ONNX 运行时（`ort`）或任意 OpenAI 兼容的
   `/embeddings` 端点。

每条管线都有配置指纹。启动时若指纹不匹配，服务端拒绝启动，除非传入
`--reindex`。解析器、分块器与嵌入器的具体设置见
[文件管线指南](/zh/docs/tutorial/file-pipeline/)。

每个分块都保存 `x-vf-src-*` 溯源元数据——来源 URI、来源名称、模型、模态，
以及分块文本本身（可用 `store_chunk_text = false` 关闭存储）。

## 锁 {#locking}

文件层对每个桶只允许一个写者。写者在 `X-vf/` 里持有 TTL 心跳的 `LOCK`
对象；写者崩溃后锁过期，别的写者可以接管。`vectorfs mcp` 以单用户写者运行，
而 `vf mcp` 以远程代理方式连接守护进程，自己不持锁。

## 对账与 GC {#gc}

对账器（reconciler）持续比对源对象与派生索引。垃圾回收只删除 key 匹配派生
分块格式**且**源对象已经不存在的向量。通过 s3vectors 面写入的向量永不被
触碰。

## 接下来 {#next}

- 亲自跑一遍：[快速开始](/zh/docs/get-started/quickstart/)
- 所有命令：[命令参考](/zh/docs/reference/commands/)
- 接入 SDK 或 Agent：[S3 Vectors 模拟器](/zh/docs/tutorial/emulator/)、[MCP](/zh/docs/tutorial/mcp/)
