---
title: 配置
description: config.toml、vectorfs.toml、环境变量与优先级。
weight: 20
---

VectorFS 区分两个文件：由 `vf config init` 写入的**客户端**配置
`~/.config/vectorfs/config.toml`，以及由 `vectorfs init` 写入数据目录的
**服务端**配置 `vectorfs.toml`。

## 优先级 {#precedence}

一条链决定所有设置：

```text
--flag  >  VF_* 环境变量  >  文件  >  内置默认值
```

在 Docker 里，`VF_*` 变量映射到标量项——端点、凭据、预设、后端。结构性
内容一律放 TOML。

## 客户端：`config.toml` {#client-config}

```bash
vf config init --endpoint http://127.0.0.1:8787 \
  --access-key <AK> --secret-key <SK>
```

文件保存端点与 SigV4 凭据。`VF_ENDPOINT`（或 `--endpoint`）可按次调用覆盖——
CI 常用这招。

## 服务端：`vectorfs.toml` {#server-config}

### `bucket_suffix` {#bucket-suffix}

```toml
bucket_suffix = "-vf"     # 默认值；安装期常量
```

分隔系统桶与源桶的后缀。在 `init` 时固定；事后改动会让既有系统桶变成孤儿。

### `[embedder.<name>]` {#embedder}

```toml
[embedder.text]
kind = "ort"                       # "ort"（本地 ONNX）或 "openai"
model = "gte-multilingual-base"    # text-rag 预设下的默认模型
# path = "/models/gte.onnx"        # 预置的模型文件
# offline = true                   # 离线：绝不拉取
# endpoint = "https://…/v1"        # OpenAI 兼容嵌入器
# dim = 1024                       # openai 需声明维度
```

### `[extractor.<mime>]` {#extractor}

```toml
[extractor."application/pdf"]
kind = "pdf-text"
```

按 MIME 定义的解析器实例。管线按名引用；见 `[ingest.<bucket>]` 的 `extract`。

### `[ingest.<bucket>]` {#ingest}

```toml
[ingest.docs]
extract = ["pdf-inspector", "pdf-text"]     # 实例名 | 有序数组 | "auto"
chunk = { kind = "token-window", size = 512, overlap = 64 }
embedders = ["text"]
merge = "rrf"                               # rrf | weighted | first
# index-prefix = "v2"                       # 可选
# include = ["**/*.pdf"]                    # 可选 glob；都省略 = 整个桶
# exclude = ["**/*.tmp"]
```

`"auto"` 只可独占 `extract` 整值。`store_chunk_text`（默认开启）决定分块文本
是否作为元数据保留。

### 非 TOML 的运行时设置 {#runtime}

以下以 flag 而非键的形式提供：`--no-auth`、`--backend`（`file`、`memory`、
`s3+http(s)://host:port`）、`max_file_size`（默认 200 MB）、扫描间隔
（默认 30 秒）、重试次数（默认 3）。

## 环境变量 {#env}

| 变量 | 作用 |
| --- | --- |
| `VF_ENDPOINT` | 客户端端点；覆盖 `config.toml` |
| `VF_*`（通用） | 覆盖对应的标量设置 |
| `HF_ENDPOINT` | 模型下载镜像（sha256 校验） |
| `AWS_ENDPOINT_URL` | 指向 VectorFS 的 AWS SDK 会读取 |

## 校验 {#validating}

`vectorfs pipelines check [--config FILE]` 按退出码语义校验管线配置——把它接进
CI。`vectorfs doctor` 还会额外探查真实后端与模型就绪情况。
