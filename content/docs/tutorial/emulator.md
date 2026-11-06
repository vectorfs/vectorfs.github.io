---
title: S3 Vectors Emulator
description: Point unmodified boto3 and aws-sdk-go-v2 at a local VectorFS for development and CI.
weight: 20
---

The s3vectors face implements all 21 operations of `s3vectors-2025-07-15` with AWS byte-compatible errors. That makes a local VectorFS a drop-in emulator for code that will eventually talk to the real S3 Vectors service — no mocks, no rewritten SDK calls.

## Start an emulator {#start}

For tests you want the volatile, unauthenticated profile:

```bash
vectorfs serve --backend memory --no-auth
```

Or with the Docker compose `emulator` profile:

```bash
docker compose -f deploy/docker-compose.yml --profile emulator up
```

The `dev` preset prepares exactly this shape: protocol only, no model download, no file layer.

## boto3 (Python) {#boto3}

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

With `--no-auth` any credentials satisfy SigV4; drop them in real deployments and use the credentials `vectorfs init` generated.

## aws-sdk-go-v2 (Go) {#go}

Set the SDK's base endpoint resolver to your VectorFS instance, then use the s3vectors client as usual:

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

Both official SDKs honor the `AWS_ENDPOINT_URL` environment variable, which is often the least invasive way to flip an existing test suite onto the emulator:

```bash
export AWS_ENDPOINT_URL=http://127.0.0.1:8787
pytest tests/
```

## Error-shape fidelity {#errors}

Compatibility extends to failures: responses use the same HTTP codes (`400`, `404`, `409`, `402`, `429`), the `X-Amzn-Errortype` header, and the `__type` field in the body. Code that catches `ClientError` from boto3 behaves identically against VectorFS and against AWS. The 32-step conformance suites in the repository's `conformance/` directory assert this for both SDKs — see [Build from source](/docs/develop/build-from-source/) for running them.

## What the emulator does not simulate {#limits}

Structural limits match AWS exactly (see the [limits reference](/docs/reference/limits/)); QPS throttling and vector-count quotas are not enforced. Behavior under real throttling still needs a staging environment.
