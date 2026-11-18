# AWS Lambda Timeout Errors

A Lambda timeout occurs when an invocation runs longer than the function's configured timeout. Lambda terminates the invocation when the timeout is reached.

## Common symptoms

CloudWatch Logs may show:

```text
Task timed out after 30.00 seconds
```

The `Duration` metric may approach the configured timeout.

## Common causes

- Slow database queries.
- Downstream API latency or unavailable services.
- Network routing, DNS, or security group problems.
- Large payload processing.
- Inefficient code or loops.
- Insufficient memory for a CPU-heavy workload.
- Connection retries without bounded timeouts.

## Step 1: Check configured timeout

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query '{Timeout:Timeout,MemorySize:MemorySize}'
```

## Step 2: Inspect logs

```bash
aws logs tail /aws/lambda/demo-lambda \
  --since 1h \
  --follow
```

Look for the last successful log line before the timeout.

## Step 3: Check duration metrics

```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=demo-lambda \
  --start-time 2026-11-18T00:00:00Z \
  --end-time 2026-11-18T01:00:00Z \
  --period 300 \
  --statistics Average Maximum
```

Replace the example time range with the incident window.

## Step 4: Set bounded network timeouts

Example with Python:

```python
import urllib.request

def lambda_handler(event, context):
    with urllib.request.urlopen(
        "https://example.com/health",
        timeout=5
    ) as response:
        return {"statusCode": response.status}
```

Use appropriate connection and read timeouts for the HTTP or database client in your application.

## Step 5: Tune memory or timeout

If the work legitimately needs more time, update the timeout:

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --timeout 60
```

If the function is CPU-bound, benchmark a larger memory setting because Lambda memory also affects available CPU.

## Best practices

- Set bounded timeouts for every external call.
- Keep downstream timeouts below the Lambda timeout.
- Monitor p95 and p99 duration.
- Optimize slow queries and avoid unnecessary network calls.
- Use asynchronous processing for work that does not need to complete in the request path.
- Do not increase the timeout simply to hide a stuck dependency.
