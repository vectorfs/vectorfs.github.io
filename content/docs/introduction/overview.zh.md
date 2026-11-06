---
title: 项目概览
description: VectorFS 做什么、服务谁、往哪里走。
weight: 10
---

VectorFS 是一个自托管、单二进制的系统，把 **S3 Vectors 兼容的向量存储** 与
**文件向量化** 结合在一起：文件放进去，语义搜索的结果出来。在本地或内存
模式下，它还可以充当 S3 Vectors 协议的开发与 CI 模拟器。

## 要解决的问题 {#problem}

做语义搜索通常意味着一个选择：要么单独跑一个向量数据库，并写代码让它与
你的文件保持同步；要么在存储上外挂一套索引服务。无论哪种，都多了一个需要
部署、监控、备份和解释的系统。

VectorFS 去掉了这第二个系统。它没有自研存储引擎——你的数据以 Lance 和
JSON 的形式，留在你已经在运行的后端里。

## 你能得到什么 {#outcome}

采用 VectorFS 之后，你可以：

- 把未修改的 AWS SDK（`boto3`、`aws-sdk-go-v2`）指向本地端点，使用完整的
  s3vectors 协议。
- 把 PDF、Markdown 或纯文本文件放进桶里，按语义查询，自己一行管线代码都不用写。
- 通过 `x-vf-src-*` 溯源元数据，把每条结果追溯到它的源文件。
- 以上一切都在一台机器上完成：一个服务端二进制，一个客户端二进制。

## 两张 API 面 {#two-api-faces}

- **s3vectors API** 实现 `s3vectors-2025-07-15` 的全部 21 个操作，错误形状
  与 AWS 逐字节兼容。
- **vectorfs API** 在 `/x-vf/<Op>` 下提供 8 个原生操作：文件进，语义搜索出。

两张 API 面都使用 SigV4 鉴权，并共享同一套错误模型。它们如何组合，见
[架构](/zh/docs/introduction/architecture/)。

## 状态与路线图 {#status}

> [!NOTE]
> v0.2 是预发布版本。文本模态是当前的 MVP：PDF、Markdown 与纯文本。

| 里程碑 | 范围 |
| --- | --- |
| v0.2（当前） | 文本模态 MVP、双 SDK 一致性套件、预设、Docker、MCP |
| v0.3（计划） | 图片模态、OCR、docx 与 HTML 解析、混合全文检索 |

## 非目标 {#non-goals}

- 不执行 QPS 限流和向量总量配额；只按 AWS 的结构性限额执行，且行为一致。
- VectorFS 不重新发明存储。持久性、副本与扩展属于你选择的后端（`file`、
  `memory` 或 `s3`）。
- 文本之外的完整多模态要等 v0.3。

## 许可证 {#license}

VectorFS 服务端与客户端采用 Apache-2.0 许可证。本网站采用 MIT 许可证。
