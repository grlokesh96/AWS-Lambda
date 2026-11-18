# AWS Lambda with Amazon EventBridge

## 1. Introduction

Amazon EventBridge is an event bus service that receives events from AWS services, your applications, and supported SaaS partners.

It can route matching events to a Lambda function.

EventBridge can also run scheduled tasks using schedules.

## 2. Architecture

```text
AWS service / Application
          |
          v
     EventBridge
          |
          | Event rule or schedule
          v
    Lambda Function
          |
          v
   Perform automation
          |
          v
    CloudWatch Logs
```

## 3. Common use cases

- Run EC2 start/stop automation on a schedule.
- Respond to AWS service events.
- Trigger cleanup scripts.
- Send alerts.
- Start periodic maintenance jobs.
- Route application events to different targets.

## 4. Example: Scheduled Lambda execution

Suppose you want a Lambda function to run every day.

You can create an EventBridge schedule and configure the Lambda function as its target.

Example schedule expression:

```text
cron(30 2 * * ? *)
```

This is a UTC schedule expression. It runs daily at 02:30 UTC. Convert your desired local time to UTC before configuring it.

For new scheduled workloads, AWS recommends EventBridge Scheduler where its features are suitable.

## 5. Create a schedule using the Console

1. Open Amazon EventBridge.
2. Open **Scheduler**.
3. Choose **Create schedule**.
4. Enter a schedule name.
5. Select a recurring schedule.
6. Enter a rate or cron expression.
7. Select Lambda as the target.
8. Choose the Lambda function.
9. Configure the execution role.
10. Review and create the schedule.

## 6. Example Python Lambda function

```python
from datetime import datetime, timezone

def lambda_handler(event, context):
    print("Scheduled Lambda execution started")

    return {
        "statusCode": 200,
        "message": "Automation completed",
        "time": datetime.now(timezone.utc).isoformat()
    }
```

## 7. EventBridge rule vs EventBridge Scheduler

| Feature | EventBridge rule | EventBridge Scheduler |
|---|---|---|
| Event routing | Yes | Not its primary purpose |
| Scheduled invocation | Supported | Yes |
| Flexible schedule features | Limited compared with Scheduler | Supports advanced scheduling options |
| Common use | Route matching events | Schedule one-time or recurring tasks |

## 8. Permissions

The configured EventBridge target needs permission to invoke the Lambda function.

For Scheduler, configure an execution role that allows it to invoke the target.

## 9. DevOps use case: EC2 automation

```text
EventBridge schedule
        |
        v
Lambda function
        |
        v
Boto3 EC2 API
        |
        v
Start / Stop EC2 instances
```

The Lambda execution role needs only the required EC2 permissions, scoped as tightly as possible.

## 10. Interview questions

**Q1. What is EventBridge?**

It is an AWS event bus service that routes events to targets based on rules. EventBridge Scheduler also supports scheduled invocations.

**Q2. Can EventBridge trigger Lambda on a schedule?**

Yes. You can use EventBridge Scheduler or scheduled EventBridge rules.

**Q3. What is a cron expression?**

It defines when a scheduled task should run.

## Summary

EventBridge is useful for event-driven automation and scheduled Lambda execution.
