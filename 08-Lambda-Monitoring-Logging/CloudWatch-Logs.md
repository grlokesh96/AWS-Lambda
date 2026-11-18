# AWS Lambda CloudWatch Logs

AWS Lambda sends function logs to Amazon CloudWatch Logs when the execution role has the required logging permissions.

## Default log group

The default log group is:

```text
/aws/lambda/<function-name>
```

Each execution environment can write to a log stream within that group.

## Required permissions

The execution role commonly needs the actions included in the AWS-managed `AWSLambdaBasicExecutionRole` policy:

- `logs:CreateLogGroup`
- `logs:CreateLogStream`
- `logs:PutLogEvents`

Review the managed policy and use a custom least-privilege policy if your security requirements call for it.

## Python logging example

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(json.dumps({
        "message": "Request received",
        "request_id": context.aws_request_id
    }))

    try:
        result = {"status": "success"}
        logger.info(json.dumps({"message": "Request completed"}))
        return result
    except Exception:
        logger.exception("Request failed")
        raise
```

Do not log the full event blindly: events may contain credentials, tokens, or personal data.

## View logs using AWS CLI

List log groups:

```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/
```

List streams for a function:

```bash
aws logs describe-log-streams \
  --log-group-name /aws/lambda/demo-lambda \
  --order-by LastEventTime \
  --descending
```

Read recent events:

```bash
aws logs tail /aws/lambda/demo-lambda \
  --since 1h \
  --follow
```

## CloudWatch Logs Insights query

Open CloudWatch Logs Insights, select the Lambda log group, and run:

```sql
fields @timestamp, @message
| sort @timestamp desc
| limit 50
```

Find error messages:

```sql
fields @timestamp, @message
| filter @message like /ERROR|Exception|Task timed out/
| sort @timestamp desc
| limit 100
```

## Configure log retention

Example: retain logs for 30 days.

```bash
aws logs put-retention-policy \
  --log-group-name /aws/lambda/demo-lambda \
  --retention-in-days 30
```

Choose retention based on operational, compliance, and cost requirements.

## Best practices

- Use structured JSON logs for easier querying.
- Include the Lambda request ID and a correlation ID where available.
- Set log retention explicitly.
- Avoid logging secrets and sensitive payloads.
- Use log levels consistently.
- Create metric filters only when a metric cannot be obtained directly from native Lambda metrics.
