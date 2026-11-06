---
title: 从源码构建
description: 工具链、构建 profile 与一致性套件。
weight: 10
---

## 工具链 {#toolchain}

| 要求 | 版本 |
| --- | --- |
| Rust | 1.94 或更高 |
| C 工具链 | 可用的 `cc`，用于原生依赖 |
| `cmake` | 用于 pdfium 等原生构建 |
| `protoc` | 编译 protocol buffer |

## 构建 {#build}

```bash
git clone https://github.com/vectorfs/vectorfs.git
cd vectorfs
cargo build --profile release-prod --workspace
```

`release-prod` 是生产 profile——发布二进制与 Docker 镜像用的就是它。日常开发
直接 `cargo build` 即可。

## 一致性套件 {#conformance}

与 AWS 的字节级兼容不是愿景，而是被断言的事实。`conformance/` 目录包含面向
两个 SDK 的 32 步套件：

- **boto3（Python）** —— 针对 s3vectors 面的 32 步，含错误形状。
- **aws-sdk-go-v2（Go）** —— 来自第二个 SDK 家族的同等覆盖。

对着新构建的服务端运行；这是确认某次协议改动没有悄悄弄坏 AWS 客户端的最快
信号。套件也通过 `vf` 与 MCP 工具覆盖 `/x-vf/` 操作。

## 测试你的改动 {#testing}

- `vectorfs pipelines check` —— 按退出码语义校验管线配置；足够廉价，可当
  pre-commit 钩子用。
- `vectorfs doctor` —— 探查真实后端的条件写、multipart 支持、模型就绪、磁盘
  空间与引擎漂移；存储层改动后尤其有用。
- emulator profile（`--backend memory --no-auth`）为套件提供易失的服务端，
  运行之间无需清理。

## 报告问题 {#issues}

请到 `github.com/vectorfs/vectorfs/issues` 提交问题。协议兼容性相关的报告，
请附上 SDK 及其版本、操作名——一致性套件大多能直接复现。
