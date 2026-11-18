# AWS Lambda Timeout Configuration

The Lambda timeout is the maximum amount of time a function is allowed to run for a single invocation. If execution exceeds the configured timeout, Lambda terminates the invocation.

## Set timeout

Example: set timeout to 30 seconds.

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --timeout 30
```

Inspect the configured timeout:

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query '{FunctionName:FunctionName,Timeout:Timeout}'
```

The maximum timeout and valid configuration range are defined by current Lambda quotas. Check AWS documentation for the current limit.

## How to choose a timeout

Consider:

- Normal and peak execution duration.
- p95 and p99 latency.
- Downstream API and database timeouts.
- Retry behavior and event-source semantics.
- The end-to-end request timeout of the caller.

Set the Lambda timeout high enough for legitimate work, but avoid making it unnecessarily long.

## Example: align downstream timeout

```python
import urllib.request

def lambda_handler(event, context):
    # Set a network timeout lower than the Lambda timeout.
    with urllib.request.urlopen(
        "https://example.com/health",
        timeout=5
    ) as response:
        return {"statusCode": response.status}
```

Use suitable connection and read timeouts for your HTTP library and downstream service.

## Timeout troubleshooting

When logs show `Task timed out`:

1. Check the `Duration` metric and `REPORT` log line.
2. Identify slow external calls.
3. Check VPC routing, DNS, and security groups if networking is involved.
4. Review database connection pooling and query performance.
5. Optimize code or adjust memory if CPU-bound.
6. Increase timeout only when the work legitimately needs more time.

## Best practices

- Use bounded timeouts for all network calls.
- Keep downstream timeouts below the Lambda timeout.
- Monitor duration percentiles and timeout errors.
- Avoid using a large timeout to hide a stuck dependency.
- Make processing idempotent when retries are possible.
