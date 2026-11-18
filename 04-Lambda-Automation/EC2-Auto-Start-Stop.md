# EC2 Auto Start and Stop Automation

## 1. Introduction

AWS Lambda can start or stop EC2 instances automatically.

For example, a DevOps team may stop development instances after working hours and start them before the team begins work.

This can reduce compute costs for instances that do not need to run continuously.

## 2. Architecture

```text
EventBridge Scheduler
        |
        | Scheduled event
        v
   Lambda Function
        |
        | Python + Boto3
        v
    Amazon EC2
        |
        v
 Start / Stop instances
        |
        v
   CloudWatch Logs
```

## 3. How it works

1. Create an EventBridge schedule.
2. Configure the schedule to invoke Lambda.
3. Lambda reads approved instance IDs or resource tags.
4. Boto3 calls the EC2 start or stop API.
5. Lambda logs the result in CloudWatch.

## 4. Example: Start EC2 instances

Create `lambda_function.py`:

```python
import boto3
import os

ec2 = boto3.client("ec2")

def lambda_handler(event, context):
    instance_ids = os.environ["INSTANCE_IDS"].split(",")

    response = ec2.start_instances(
        InstanceIds=[item.strip() for item in instance_ids]
    )

    print("Start request submitted:", response["StartingInstances"])
    return {"message": "Start request submitted"}
```

Set the `INSTANCE_IDS` environment variable to approved instance IDs, separated by commas.

## 5. Example: Stop EC2 instances

```python
import boto3
import os

ec2 = boto3.client("ec2")

def lambda_handler(event, context):
    instance_ids = os.environ["INSTANCE_IDS"].split(",")

    response = ec2.stop_instances(
        InstanceIds=[item.strip() for item in instance_ids]
    )

    print("Stop request submitted:", response["StoppingInstances"])
    return {"message": "Stop request submitted"}
```

These examples submit state-change requests; they do not wait until instances reach the final state.

## 6. Required IAM permissions

For a simple fixed-instance example, the Lambda execution role needs the relevant actions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

This is a learning example. EC2 start/stop permissions can be restricted using supported resource-level permissions and conditions. Scope them to approved instances wherever possible.

## 7. Configure EventBridge Scheduler

Create two schedules:

- Start schedule: invokes the start function before working hours.
- Stop schedule: invokes the stop function after working hours.

Configure the correct time zone in EventBridge Scheduler. Do not assume cron expressions use your local time zone.

## 8. Safer production design

- Use tags such as `AutoStop=true`.
- Exclude production and critical instances.
- Use an allowlist of approved instance IDs or tags.
- Check instance state before requesting an action.
- Add a dry-run mode.
- Notify owners when an action is taken.
- Handle instances that cannot be stopped.
- Consider Auto Scaling groups and application dependencies.

## 9. Troubleshooting

| Problem | What to check |
|---|---|
| AccessDenied | Lambda execution role permissions |
| Function not triggered | EventBridge schedule and target role |
| Instance not stopping | Instance state, protection settings, and permissions |
| Wrong instances affected | Environment variables, tags, and allowlist |
| No logs | CloudWatch Logs permissions and log group |

## 10. Interview questions

**Q1. How can Lambda start and stop EC2 instances?**

Use Boto3 EC2 APIs such as `start_instances()` and `stop_instances()`, triggered by EventBridge Scheduler.

**Q2. Why use EventBridge Scheduler?**

It invokes Lambda at configured times without requiring a server or cron host.

**Q3. How do you prevent production instances from being stopped?**

Use explicit allowlists, resource tags, account/region controls, and deny-by-default logic.

## Summary

Lambda and EventBridge can automate EC2 start/stop operations. Always test the target selection and permissions before enabling automatic actions.
