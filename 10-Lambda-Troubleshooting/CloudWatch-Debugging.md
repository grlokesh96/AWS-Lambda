# Debugging AWS Lambda with CloudWatch

Amazon CloudWatch provides Lambda logs, metrics, alarms, and Logs Insights queries. These tools are often the first place to investigate a failed invocation.

## 1. Find the Lambda log group

Default log group:

```text
/aws/lambda/<function-name>
```

List Lambda log groups:

```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/
```

## 2. Tail recent logs

```bash
aws logs tail /aws/lambda/demo-lambda \
  --since 1h \
  --follow
```

Look for request IDs, exception messages, stack traces, and `REPORT` lines.

## 3. Search errors with Logs Insights

```sql
fields @timestamp, @message
| filter @message like /ERROR|Exception|Task timed out|AccessDenied/
| sort @timestamp desc
| limit 100
```

Find Lambda report lines:

```sql
fields @timestamp, @message
| filter @message like /REPORT RequestId/
| sort @timestamp desc
| limit 50
```

Find a particular request ID:

```sql
fields @timestamp, @message
| filter @message like /REQUEST_ID_VALUE/
| sort @timestamp asc
```

Replace `REQUEST_ID_VALUE` with the actual invocation request ID.

## 4. Review key metrics

In the `AWS/Lambda` namespace, check:

- `Invocations`
- `Errors`
- `Duration`
- `Throttles`
- `ConcurrentExecutions`

For asynchronous or stream workloads, also review relevant event age and dropped-event metrics.

## 5. Check function configuration

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query '{State:State,LastUpdateStatus:LastUpdateStatus,Runtime:Runtime,Handler:Handler,MemorySize:MemorySize,Timeout:Timeout,Role:Role}'
```

This helps identify configuration changes and deployment state.

## 6. Correlate logs with deployments

Compare incident timestamps with:

- CI/CD deployment history.
- Lambda code and configuration updates.
- IAM policy changes.
- VPC, security group, or route changes.
- Downstream database and API incidents.

## Structured logging example

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(json.dumps({
        "message": "Request started",
        "request_id": context.aws_request_id
    }))
    return {"statusCode": 200}
```

Never log credentials, access tokens, or sensitive payloads.

## Best practices

- Set log retention to control cost and meet retention requirements.
- Use structured logs and correlation IDs.
- Create alarms for important failure signals.
- Enable tracing when cross-service latency needs investigation.
- Preserve logs and metrics during incident response.
- Turn findings into a repeatable runbook and automated alert.
