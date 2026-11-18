# Project 03: S3 File Processing

## Objective

Process files automatically when they are uploaded to an Amazon S3 bucket. Example use cases include image metadata extraction, CSV validation, document conversion, and data ingestion.

## Architecture

```text
Client uploads file
        |
        v
S3 bucket: incoming/
        |
   ObjectCreated event
        |
        v
Lambda processor
        |
        +----> Validate / transform
        |
        v
S3 bucket: processed/
        |
        v
CloudWatch Logs + failure destination
```

## Avoid recursive invocation

Use separate input and output buckets, or configure the notification only for an `incoming/` prefix. If Lambda writes output to the same bucket and the event filter includes that output path, the function can invoke itself repeatedly.

## Lambda example: process a text or CSV object

```python
import boto3
import csv
import io
import urllib.parse

s3 = boto3.client("s3")

def lambda_handler(event, context):
    results = []

    for record in event.get("Records", []):
        bucket = record["s3"]["bucket"]["name"]
        key = urllib.parse.unquote_plus(record["s3"]["object"]["key"])

        if not key.startswith("incoming/"):
            continue

        response = s3.get_object(Bucket=bucket, Key=key)
        content = response["Body"].read().decode("utf-8")

        # Example validation: parse CSV and count rows.
        rows = list(csv.reader(io.StringIO(content)))
        row_count = max(len(rows) - 1, 0)  # assumes first row is a header

        output_key = key.replace("incoming/", "processed/", 1) + ".result.txt"
        s3.put_object(
            Bucket=bucket,
            Key=output_key,
            Body=f"File: {key}\nData rows: {row_count}\n".encode("utf-8"),
            ContentType="text/plain"
        )
        results.append({"source": key, "output": output_key, "rows": row_count})

    return {"processed": results}
```

For large files, avoid loading the entire object into memory. Use streaming, chunked processing, or a service designed for large-scale data processing. Configure Lambda memory, timeout, and ephemeral storage based on measured file sizes.

## S3 notification configuration

1. Create an S3 bucket.
2. Create a Lambda function and execution role.
3. Add an S3 trigger for `ObjectCreated`.
4. Configure prefix `incoming/` and, if useful, a suffix such as `.csv`.
5. Allow S3 to invoke the Lambda function.
6. Upload a sample file and inspect the logs.

## IAM permissions

The Lambda role needs `s3:GetObject` for the input prefix and `s3:PutObject` for the processed prefix. The bucket policy must also permit the required access. Keep access scoped to the bucket and prefixes; account for KMS permissions if objects use SSE-KMS.

## Reliability and security

- S3 event notifications are at-least-once and can arrive out of order. Make processing idempotent.
- Use the object key and version ID (when versioning is enabled) to identify a specific input.
- For high-volume or retry-sensitive workloads, route events through SQS or EventBridge.
- Validate file type and content; do not trust filename extensions.
- Treat uploaded files as untrusted input and avoid executing uploaded content.
- Configure failure handling and alarms.
