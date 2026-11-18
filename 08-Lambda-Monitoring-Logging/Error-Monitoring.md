# AWS Lambda Error Monitoring

Error monitoring helps detect failures, identify root causes, and reduce the impact of Lambda application incidents.

## Common Lambda errors

| Error | Possible cause |
|---|---|
| `Runtime.ExitError` | Runtime or process exited unexpectedly |
| `Task timed out` | Execution exceeded configured timeout |
| `OutOfMemory` / memory-related failure | Function memory limit is insufficient or memory use is excessive |
| `AccessDeniedException` | Missing IAM, resource, or KMS permission |
| `ThrottlingException` | Concurrency or service quota pressure |
| `ImportModuleError` | Incorrect handler, package layout, or missing dependency |
| `ECONNRESET` / connection timeout | Network or downstream service issue |

## Investigation workflow

```text
Alert Received
      |
      v
Check CloudWatch Metrics
      |
      v
Find Failed Invocation in Logs
      |
      v
Inspect Request ID and Stack Trace
      |
      v
Check IAM / Network / Dependencies
      |
      v
Identify Root Cause
      |
      v
Fix, Test, Deploy, and Verify
```

## Find errors in CloudWatch Logs Insights

```sql
fields @timestamp, @message
| filter @message like /ERROR|Exception|Task timed out/
| sort @timestamp desc
| limit 100
```

## Check Lambda metrics

```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=demo-lambda \
  --start-time 2026-11-18T00:00:00Z \
  --end-time 2026-11-18T01:00:00Z \
  --period 300 \
  --statistics Sum
```

Use the actual incident time range and Region.

## Improve error handling

Example:

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Application logic
        return {"status": "success"}
    except ValueError:
        logger.exception("Invalid input")
        raise
    except Exception:
        logger.exception("Unexpected application failure")
        raise
```

Re-raising the exception allows Lambda to record the invocation as an error and lets configured retry or failure-destination behavior work as expected. Handle expected business outcomes separately when they should not be treated as invocation failures.

## Operational best practices

- Include request IDs and correlation IDs in logs.
- Use alarms for errors, throttles, and latency.
- Monitor asynchronous event age and stream iterator age where applicable.
- Configure retries, dead-letter queues, or destinations according to event source semantics.
- Make handlers idempotent when events can be retried.
- Correlate incidents with deployments and downstream service health.
- Create runbooks for common failure patterns.
