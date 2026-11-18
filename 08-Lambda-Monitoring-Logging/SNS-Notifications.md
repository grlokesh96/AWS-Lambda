# Amazon SNS Notifications for Lambda Monitoring

Amazon Simple Notification Service (SNS) can deliver CloudWatch alarm notifications to email, SMS, or other supported subscribers and integrations.

## Architecture

```text
Lambda Function
      |
      v
CloudWatch Metrics
      |
      v
CloudWatch Alarm
      |
      v
SNS Topic
      |
      +---- Email
      +---- SMS (where supported)
      +---- HTTPS endpoint / integration
```

## 1. Create an SNS topic

```bash
aws sns create-topic \
  --name lambda-alerts \
  --region ap-south-1
```

Record the topic ARN returned by the command.

## 2. Subscribe an email address

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:123456789012:lambda-alerts \
  --protocol email \
  --notification-endpoint alerts@example.com \
  --region ap-south-1
```

The recipient must confirm the subscription using the confirmation email before notifications are delivered.

## 3. Publish a test message

```bash
aws sns publish \
  --topic-arn arn:aws:sns:ap-south-1:123456789012:lambda-alerts \
  --subject "Lambda monitoring test" \
  --message "This is a test notification." \
  --region ap-south-1
```

## 4. Connect a CloudWatch alarm

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

CloudWatch publishes alarm state notifications to the SNS topic. Ensure the topic policy allows the CloudWatch alarm service to publish, following the current AWS guidance for your Region and account.

## 5. Test alarm delivery

You can set a test threshold in a non-production environment or use CloudWatch alarm test mechanisms where available. Confirm the subscription, topic policy, alarm action, and recipient delivery.

## Security best practices

- Restrict SNS topic publishing and subscription permissions.
- Use encryption at rest when required by your security policy.
- Do not include passwords, tokens, or sensitive event payloads in notifications.
- Keep the subscriber list current.
- Route critical alerts to an actively monitored on-call channel.
- Define severity, ownership, and a runbook for each alert.
- Avoid alert fatigue by tuning thresholds and using composite alarms when useful.
