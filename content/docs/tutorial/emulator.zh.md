---
title: S3 Vectors 模拟器
description: 把未修改的 boto3 与 aws-sdk-go-v2 指向本地 VectorFS，用于开发与 CI。
weight: 20
---

s3vectors 面实现 `s3vectors-2025-07-15` 的全部 21 个操作，错误与 AWS 逐字节
兼容。这让本地 VectorFS 成为一个可直接替换的模拟器——将来要连真实 S3 Vectors
服务的代码现在就能在本地跑通：不需要 mock，也不用改写 SDK 调用。

## 启动模拟器 {#start}

测试场景要用易失、免鉴权的形态：

```bash
vectorfs serve --backend memory --no-auth
```

或使用 Docker compose 的 `emulator` profile：

```bash
docker compose -f deploy/docker-compose.yml --profile emulator up
```

`dev` 预设准备的就是这个形态：仅协议、不下载模型、不启用文件层。

## boto3（Python） {#boto3}

```python
import boto3

client = boto3.client(
    "s3vectors",
    endpoint_url="http://127.0.0.1:8787",
    aws_access_key_id="dev",
    aws_secret_access_key="dev",
    region_name="us-east-1",
)

client.create_vector_bucket(vectorBucketName="docs")
client.create_index(
    vectorBucketName="docs",
    indexName="chunks",
    dataType="float32",
    dimension=1024,
    distanceMetric="cosine",
)
client.put_vectors(
    vectorBucketName="docs",
    indexName="chunks",
    vectors=[{"key": "a", "data": {"float32": [1.0, 0.0, 0.0]}}],
)
resp = client.query_vectors(
    vectorBucketName="docs",
    indexName="chunks",
    queryVector={"float32": [1.0, 0.0, 0.0]},
    topK=2,
    returnMetadata=True,
)
```

使用 `--no-auth` 时任意凭据都能通过 SigV4；正式部署请去掉该选项，改用
`vectorfs init` 生成的凭据。

## aws-sdk-go-v2（Go） {#go}

把 SDK 的 base endpoint 解析器指向你的 VectorFS 实例，之后照常使用 s3vectors
客户端：

```go
cfg, err := config.LoadDefaultConfig(ctx)
if err != nil {
    return err
}
client := s3vectors.NewFromConfig(cfg, func(o *s3vectors.Options) {
    o.BaseEndpoint = aws.String("http://127.0.0.1:8787")
})

out, err := client.QueryVectors(ctx, &s3vectors.QueryVectorsInput{
    VectorBucketName: aws.String("docs"),
    IndexName:        aws.String("chunks"),
    QueryVector:      &types.QueryVector{Float32: []float32{1.0, 0.0, 0.0}},
    TopK:             aws.Int32(2),
})
```

## AWS_ENDPOINT_URL {#endpoint-url}

两个官方 SDK 都支持 `AWS_ENDPOINT_URL` 环境变量，这通常是把现成测试套件切到
模拟器上侵入性最小的办法：

```bash
export AWS_ENDPOINT_URL=http://127.0.0.1:8787
pytest tests/
```

## 错误形状的保真度 {#errors}

兼容性延伸到失败路径：响应使用相同的 HTTP 状态码（`400`、`404`、`409`、
`402`、`429`）、相同的 `X-Amzn-Errortype` 头，以及 body 中的 `__type` 字段。
捕获 boto3 `ClientError` 的代码面对 VectorFS 与面对 AWS 的行为一致。仓库
`conformance/` 目录下的 32 步一致性套件为两个 SDK 断言了这一点——运行方式见
[从源码构建](/zh/docs/develop/build-from-source/)。

## 模拟器不模拟什么 {#limits}

结构性限额与 AWS 完全一致（见[限额参考](/zh/docs/reference/limits/)）；QPS
限流与向量配额不执行。真实限流下的行为仍需在预发环境中验证。
