# Project 01: EC2 Auto Start/Stop

## Objective

Automatically stop non-production EC2 instances outside working hours and start them before the team begins work. This reduces compute costs while keeping development environments available on schedule.

## Architecture

```text
EventBridge schedule (start / stop)
              |
              v
        Lambda function
              |
        IAM execution role
              |
              v
      EC2 instances (tagged)
```

## Prerequisites

- An AWS account and AWS CLI configured.
- One or more test EC2 instances.
- An IAM role for Lambda with scoped EC2 permissions.
- A tag such as `AutoSchedule=office-hours`.

## Lambda code (Python / Boto3)

Create `lambda_function.py`:

```python
import os
import boto3

ec2 = boto3.client("ec2")
TAG_KEY = os.environ.get("TAG_KEY", "AutoSchedule")
TAG_VALUE = os.environ.get("TAG_VALUE", "office-hours")

def lambda_handler(event, context):
    action = event.get("action", "stop").lower()
    if action not in {"start", "stop"}:
        raise ValueError("action must be start or stop")

    response = ec2.describe_instances(
        Filters=[
            {"Name": f"tag:{TAG_KEY}", "Values": [TAG_VALUE]},
            {"Name": "instance-state-name", "Values": (
                ["stopped"] if action == "start" else ["running"]
            )}
        ]
    )

    instance_ids = [
        instance["InstanceId"]
        for reservation in response["Reservations"]
        for instance in reservation["Instances"]
    ]

    if not instance_ids:
        return {"action": action, "instances": [], "message": "No matching instances"}

    if action == "start":
        ec2.start_instances(InstanceIds=instance_ids)
    else:
        ec2.stop_instances(InstanceIds=instance_ids)

    return {"action": action, "instances": instance_ids}
```

For accounts with many instances, implement pagination for `DescribeInstances` or use a paginator. Validate that the selected instances are safe to stop; exclude databases, production workloads, and instances with stop protection where relevant.

## EventBridge schedules

Create two scheduled rules (or EventBridge Scheduler schedules) that invoke the same Lambda with different JSON inputs:

Start event:

```json
{"action": "start"}
```

Stop event:

```json
{"action": "stop"}
```

Example cron expressions in UTC:

```text
cron(0 3 ? * MON-FRI *)   # 09:00 IST, start
cron(0 13 ? * MON-FRI *)  # 19:00 IST, stop
```

Adjust for your working hours and daylight-saving requirements if the schedule is tied to a region that observes daylight saving time. EventBridge rule cron schedules are UTC.

## IAM permissions

The execution role needs `ec2:DescribeInstances`, `ec2:StartInstances`, and `ec2:StopInstances`. Scope start/stop permissions to approved instance ARNs and use supported resource-tag conditions where possible. `DescribeInstances` generally requires `Resource: "*"`.

## Deploy and test

Package and deploy using the Lambda console, AWS SAM, or Terraform. Test first with:

```json
{"action": "stop"}
```

Check results:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:AutoSchedule,Values=office-hours" \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,Tags]'
```

## Production considerations

- Add an opt-out tag and explicit allow-list.
- Make the function idempotent and handle instances transitioning between states.
- Log selected instance IDs and the action taken.
- Configure an alarm for Lambda errors and failed invocations.
- Consider AWS Instance Scheduler for a managed scheduling solution.

## Verification checklist

- [ ] Only tagged test instances are selected.
- [ ] Start and stop schedules use the correct timezone conversion.
- [ ] Lambda role has only required permissions.
- [ ] A failure alert is configured.
