---
title: 环境要求
description: 硬件、操作系统与网络前提。
weight: 10
---

## 操作系统 {#os}

- **服务端（`vectorfs`）** —— Linux。Docker 镜像是多架构的，并以非 root 运行。
- **客户端（`vf`）** —— Linux 与 Windows。Windows 安装脚本只覆盖客户端。
- **Docker** —— 任何能跑 Docker 的主机都可以；见[部署指南](/zh/docs/tutorial/deploy/)。

## 硬件 {#hardware}

- **纯 CPU 即可** —— 默认的 `text-rag` 预设用本地 ONNX 运行时嵌入，不需要 GPU。
- **内存** —— 按嵌入模型加你的负载规划。默认模型 `gte-multilingual-base`
  大约需要 1.2 GiB。
- **磁盘** —— 模型下载（约 1.2 GiB）加上数据目录。索引的大小随分块数量
  增长，而不是随文件数量。

## 网络 {#network}

首次执行 `vectorfs init --preset text-rag` 会下载嵌入模型。如果直连慢或不通，
把 `HF_ENDPOINT` 指向镜像。下载内容会做 sha256 校验。

之后，`file` 后端的服务端不再有任何出站调用。`s3` 后端只与你指定的 S3 兼容
端点通信；OpenAI 兼容嵌入器只与你配置的端点通信。

## 从源码构建 {#from-source}

自行构建服务端需要 Rust 1.94 或更高版本、C 工具链、`cmake` 与 `protoc`。
见[从源码构建](/zh/docs/develop/build-from-source/)。

> [!TIP]
> 只想用现成的 boto3 测试套件试一试 s3vectors 协议？`dev` 预设完全不需要
> 下载模型。
