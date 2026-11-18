# CloudWatch Alarms for AWS Lambda

CloudWatch alarms evaluate metrics against a threshold and can notify an SNS topic or trigger supported automated actions.

## Common alarm use cases

- Lambda errors increase above an acceptable level.
- Lambda throttles occur.
- Function duration approaches the configured timeout.
- Concurrent executions approach a configured limit.

## Create an alarm for Lambda errors

This example alarms when the function records at least one error in a 5-minute period.

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name demo-lambda-errors \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=demo-lambda \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --treat-missing-data notBreaching
```

The threshold is only an example. For low-volume functions, a single error may be important; for high-volume functions, an error-rate alarm may be more meaningful.

## Create an alarm for throttles

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name demo-lambda-throttles \
  --namespace AWS/Lambda \
  --metric-name Throttles \
  --dimensions Name=FunctionName,Value=demo-lambda \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --treat-missing-data notBreaching
```

## Add an SNS alarm action

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name demo-lambda-errors \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=demo-lambda \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:ap-south-1:123456789012:lambda-alerts
```

The SNS topic must exist, and the alarm must be allowed to publish to it.

## Inspect alarm state

```bash
aws cloudwatch describe-alarms \
  --alarm-names demo-lambda-errors
```

## Alarm states

| State | Meaning |
|---|---|
| `OK` | Metric is within the configured threshold |
| `ALARM` | Threshold condition has been met |
| `INSUFFICIENT_DATA` | Not enough data is available to evaluate the alarm |

## Best practices

- Tune thresholds using actual traffic and baseline behavior.
- Consider evaluation periods to reduce noisy alerts.
- Choose `treat-missing-data` deliberately; missing data does not always mean healthy.
- Alarm on user impact, not every harmless transient event.
- Test the SNS notification path.
- Review alarms after changes to traffic, timeout, memory, or concurrency.
