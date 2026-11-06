---
title: 设计文档
description: 权威决策所在——从命令表到 v3 桶模型。
weight: 20
---

VectorFS 仓库把设计历史公开保存。当行为与宣传不一致时，以这些文档为准。

## 文档地图 {#map}

| 文档 | 内容 |
| --- | --- |
| `design.md` | 主设计文档，含附录 E —— [命令参考](/zh/docs/reference/commands/)背后的权威命令表 |
| `design-v3.md` | v3 桶模型：源桶加 `-vf` 系统桶、锁、对账 |
| `CONTEXT.md` | 术语表 —— 贯穿代码、文档与 issue 的领域语言（`source`、`chunk`、`reindex`、预设、后端） |
| `docs/adr/0001` | 命名 ADR：客户端二进制名 `vf` 与 fish 的 VirtualFish 撞名一案，已有裁决 |
| `spike/REPORT.md` | 塑造架构的预研报告 |
| `conformance/` | 双 SDK、32 步的一致性套件 |

## 决策如何记录 {#adr}

跨模块的决策以 ADR（架构决策记录）的形式存于 `docs/adr/`。ADR 0001 裁定了
客户端二进制的名字——`vf`——接受了与 fish VirtualFish 插件的撞名，并记录了
理由。遇到命名或兼容性问题，先查 ADR；解决了问题，就补一条记录。

## 术语纪律 {#terminology}

`CONTEXT.md` 是用词的事实来源：**source（源）** 是源桶里的对象（不是
"document"）；**chunk（分块）** 是检索单元，每个对应一个向量；**reindex
（重建索引）** 是管线变更后显式的全量重嵌入。代码、提交信息与 issue 都使用
这些术语——请让你的贡献保持同样的纪律。

## 本站源码 {#website}

本站位于 [`vectorfs/vectorfs.github.io`](https://github.com/vectorfs/vectorfs.github.io)
仓库，用 [OINK](https://oink.pgsty.com/) 主题基于 Hugo 构建。本地预览方法见
该仓库的 README。
