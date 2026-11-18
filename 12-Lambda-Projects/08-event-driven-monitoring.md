# Project 08: Event-Driven Monitoring

## Objective

Create an event-driven monitoring workflow that reacts to AWS service events, enriches the event, and sends an actionable notification to the operations team.

## Architecture

```text
AWS service event / CloudWatch alarm
                 |
                 v
             EventBridge
                 |
                 v
          Lambda event handler
                 |
          Enrich + classify
                 |
                 v
             SNS topic
                 |
          Email / Chat integration
```

## Example: process EC2 state-change events

Create an EventBridge rule for EC2 instance state changes. A sample event pattern:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["stopped", "terminated"]
  }
}
```

Use the EventBridge console or CLI to create the rule and attach the Lambda target. Grant EventBridge permission to invoke the function.

## Lambda notification handler

```python
import json
import os
import boto3

sns = boto3.client("sns")
TOPIC_ARN = os.environ["SNS_TOPIC_ARN"]

def lambda_handler(event, context):
    detail = event.get("detail", {})
    instance_id = detail.get("instance-id", "unknown")
    state = detail.get("state", "unknown")

    message = {
        "event": event.get("detail-type"),
        "account": event.get("account"),
        "region": event.get("region"),
        "instance_id": instance_id,
        "state": state,
        "time": event.get("time")
    }

    sns.publish(
        TopicArn=TOPIC_ARN,
        Subject=f"EC2 state changed: {instance_id}",
        Message=json.dumps(message, indent=2)
    )

    return {"notified": True, "instance_id": instance_id, "state": state}
```

## CloudWatch alarm integration

CloudWatch alarms can publish state-change notifications to SNS or send events to EventBridge. Use alarms for metric thresholds such as:
- High EC2 CPU utilization sustained over a defined period.
- Lambda `Errors` or `Throttles` above a threshold.
- API Gateway 5xx errors.
- SQS queue age or visible-message count exceeding an operational limit.

Choose thresholds based on service objectives and normal baselines, not arbitrary defaults.

## IAM permissions

The Lambda role needs `sns:Publish` scoped to the target topic and the standard CloudWatch Logs permissions. EventBridge needs permission to invoke the function. If the function queries additional resources for enrichment, grant only the required read APIs.

## Reliability and observability

- EventBridge delivery is at-least-once; design for duplicate events.
- Configure retry policies and a dead-letter queue for target delivery failures.
- Keep messages concise and include account, region, resource, timestamp, and a runbook link.
- Avoid putting secrets or sensitive payloads into email or chat messages.
- Monitor Lambda errors, duration, throttles, and asynchronous failures.
- Use correlation IDs to connect an alert with logs and remediation records.

## Extension: route by severity

Use EventBridge rules or a small routing function to send critical alerts to an on-call channel and lower-severity events to an operations digest. Add deduplication and suppression windows to reduce alert fatigue.

## Verification checklist

- [ ] EventBridge rule matches only intended event types.
- [ ] Lambda invocation permission is configured.
- [ ] SNS topic subscription is confirmed.
- [ ] Duplicate events do not cause harmful repeated actions.
- [ ] Retry/DLQ and CloudWatch alarms are configured.
- [ ] Notifications include enough context for an operator to act.
