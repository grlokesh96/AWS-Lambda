# CloudWatch Events and AWS Lambda

## 1. Introduction

CloudWatch Events was the earlier name for the event service that evolved into Amazon EventBridge.

Today, Amazon EventBridge provides event routing and scheduling capabilities. You may still see the name CloudWatch Events in older AWS documentation, tutorials, policies, and projects.

**Simple meaning:** Events from AWS services or scheduled rules can invoke Lambda functions.

## 2. Architecture

```text
AWS service event
       |
       v
EventBridge
(formerly CloudWatch Events)
       |
       | Matching rule
       v
Lambda Function
       |
       v
Perform action
       |
       v
CloudWatch Logs
```

## 3. Common use cases

- Run a function on a schedule.
- React to EC2 state changes.
- Send alerts for AWS events.
- Automate infrastructure tasks.
- Start maintenance jobs.
- Route events to different targets.

## 4. Example: Run Lambda when EC2 changes state

A DevOps team may want to send a notification when an EC2 instance changes state.

Example event pattern:

```json
{
  "source": [
    "aws.ec2"
  ],
  "detail-type": [
    "EC2 Instance State-change Notification"
  ],
  "detail": {
    "state": [
      "stopped"
    ]
  }
}
```

This pattern matches EC2 instance state-change events where the state is `stopped`.

## 5. Create an EventBridge rule

1. Open Amazon EventBridge.
2. Select **Rules**.
3. Choose **Create rule**.
4. Enter a rule name.
5. Select the event bus.
6. Choose an event pattern.
7. Select the matching AWS service and event.
8. Choose Lambda as the target.
9. Select your Lambda function.
10. Create the rule.

The rule needs permission to invoke the Lambda function. The Console can configure this permission when adding the target.

## 6. Scheduled rule example

An older scheduled EventBridge rule can use an expression such as:

```text
rate(5 minutes)
```

This schedules the target to run every five minutes.

For new scheduled tasks, consider EventBridge Scheduler, which provides additional scheduling features.

## 7. CloudWatch Events vs EventBridge

| CloudWatch Events (legacy name) | EventBridge |
|---|---|
| Original event service name | Current service name |
| Basic event rules and schedules | Event buses, rules, and richer integrations |
| Seen in older tutorials | Recommended terminology for new designs |

CloudWatch is still the name of the monitoring service. CloudWatch metrics and logs remain important for monitoring Lambda.

## 8. Troubleshooting

- Check whether the event pattern matches the event.
- Confirm the rule is enabled.
- Verify the target Lambda function.
- Check the Lambda resource-based permission.
- Review EventBridge metrics and Lambda CloudWatch Logs.

## 9. Interview questions

**Q1. What was CloudWatch Events?**

It was the earlier name for the AWS event service that evolved into EventBridge.

**Q2. Can an EC2 state change trigger Lambda?**

Yes. An EventBridge rule can match an EC2 state-change event and invoke Lambda.

**Q3. What is an event pattern?**

It is a JSON filter that determines which events match an EventBridge rule.

## Summary

For new event-driven designs, use Amazon EventBridge terminology. CloudWatch Events is mainly encountered in older tutorials and existing configurations.
