---
title: 文件管线
description: 为你的语料调整规范化、解析、分块与嵌入。
weight: 10
---

每个源对象——源桶里的一个文件——都经过同一条管线。本指南解释每个阶段以及
控制它的参数。

```text
normalize → extract → chunk → embed → index
```

## 规范化 {#normalize}

入口步骤会解开 gzip 等容器，让后续阶段面对的始终是原始字节。这里没有可配置项。

## 解析 {#extract}

解析器从源文件抽出文本。v0.2 支持：

| 格式 | 解析器 |
| --- | --- |
| PDF | 走 pdfium 的 `pdf-text`，或纯 Rust 的 `pdf-inspector` |
| Markdown | 原生 |
| 纯文本 | 原生 |

选择遵循有序回退链，并按内容路由——一个扩展名是 `.pdf` 但实为 gzip 文本的
文件，照样能正确解析。在 `vectorfs.toml` 里，每个 MIME 类型通过
`[extractor.<mime>]` 映射：

```toml
[extractor."application/pdf"]
kind = "pdf-text"

[ingest.docs]
extract = ["pdf-inspector", "pdf-text"]   # 有序回退链
```

`extract` 的值可以是单个实例名、像上面这样的有序数组，或 `"auto"` ——
`"auto"` 只可独占整值。

## 分块 {#chunk}

分块器把解析出的文本切成检索单元；每个分块对应一个向量。默认是 CJK 安全的
token 窗口策略：

```toml
[ingest.docs]
chunk = { kind = "token-window", size = 512, overlap = 64 }
```

更小的窗口对密集、分节的文档检索更精准；更大的窗口为每个命中保留更多上下文。
保留一些重叠，让跨边界的句子仍可被检索到。

## 嵌入 {#embed}

嵌入器按分块扇出。支持两种：

```toml
# 本地 ONNX 运行时（text-rag 下的默认）
[embedder.text]
kind = "ort"
model = "gte-multilingual-base"

# 任意 OpenAI 兼容端点
[embedder.openai]
kind = "openai"
model = "your-model"
endpoint = "https://api.example.com/v1"
dim = 1024                    # openai 嵌入器需声明维度
```

离线安装请预置模型并设置 `path` 与 `offline = true`。桶的 `embedders` 列表
决定运行哪些嵌入器；`merge` 决定多索引结果如何合并 —— `rrf`（默认）、
`weighted` 或 `first`。

## 溯源元数据 {#provenance}

每个分块在向量旁边保存 `x-vf-src-*` 元数据：

| 键 | 含义 |
| --- | --- |
| `x-vf-src-uri` | 源对象 URI（≤100 字节，可过滤） |
| `x-vf-src-name` | 源文件名 |
| `x-vf-model` | 所用嵌入模型 |
| `x-vf-modality` | 模态，v0.2 为 `text` |
| `x-vf-chunk-text` | 分块文本本身；用 `store_chunk_text = false` 可不存 |

因为 URI 可过滤，`vf search --filter '{"x-vf-src-uri": {"eq": "docs/report.pdf"}}'`
这样的查询可以把结果限制在某个文件或前缀内。

## 指纹与重建索引 {#reindex}

服务端为每条管线记录配置指纹。修改解析器、分块器或嵌入器后，已存指纹不再
匹配，`vectorfs serve` 会拒绝启动，而不是悄悄混用两套配置产生的向量。二选一：

```bash
vectorfs serve --reindex          # 跳过门禁，启动时重建索引
vf reindex --bucket docs          # 在运行中的守护进程上触发（异步动词）
```

`vf reindex` 只负责触发——实际执行者是守护进程。用 `vf sources status`
跟踪进度。

## 限定摄取范围 {#scoping}

`[ingest.<bucket>]` 接受 `include` 与 `exclude` glob。两者都省略则摄取整个桶；
可选的 `index-prefix` 用来按管线分隔派生索引。
