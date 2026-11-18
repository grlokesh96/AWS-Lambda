# AWS X-Ray for Lambda

AWS X-Ray helps trace requests as they move through distributed applications. It can show how much time a request spends in Lambda and supported downstream services.

## Why use X-Ray?

- Understand end-to-end request latency.
- Identify slow downstream dependencies.
- View service maps and trace segments.
- Investigate errors across service boundaries.

## Enable active tracing

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --tracing-config Mode=Active \
  --region ap-south-1
```

The Lambda execution role needs the permissions required to write trace data. The AWS-managed `AWSXRayDaemonWriteAccess` policy is a common starting point; review its permissions and current AWS guidance.

## Python example

For supported runtimes and instrumentation approaches, use the AWS X-Ray SDK or supported OpenTelemetry instrumentation as appropriate.

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(json.dumps({
        "message": "Processing request",
        "request_id": context.aws_request_id
    }))
    return {"statusCode": 200, "body": "OK"}
```

Enabling Lambda active tracing creates Lambda trace data. To trace downstream calls in detail, instrument supported clients/libraries using the recommended tracing approach for your runtime. The example above does not itself create custom subsegments.

## View traces

In the AWS Console:

1. Open AWS X-Ray or the supported tracing view in CloudWatch.
2. Select the relevant time range.
3. Open the service map.
4. Select a trace to inspect segments and latency.
5. Correlate trace timestamps and request IDs with Lambda logs.

## Sampling

X-Ray uses sampling rules to control how many requests are traced. Review sampling settings to balance observability, cost, and trace coverage. A sampled trace is not a record of every invocation.

## Best practices

- Enable tracing where it adds value to troubleshooting.
- Instrument downstream clients for useful dependency timing.
- Avoid putting secrets or sensitive payloads into trace annotations or metadata.
- Use correlation IDs to connect logs and traces.
- Monitor sampling and trace ingestion behavior.
