# EC2 Automation Script with Python and Boto3

This example shows how to list EC2 instances and start or stop a specific instance using Lambda.

## IAM permissions

For listing and starting/stopping instances, the execution role may need:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:DescribeInstances",
      "ec2:StartInstances",
      "ec2:StopInstances"
    ],
    "Resource": "*"
  }]
}
```

EC2 start/stop actions support resource-level permissions; scope them to approved instance ARNs in production. `DescribeInstances` generally requires `"Resource": "*"`.

## Python Lambda code

```python
import os
import boto3

ec2 = boto3.client("ec2", region_name=os.environ.get("AWS_REGION", "ap-south-1"))

def lambda_handler(event, context):
    action = event.get("action", "list")

    if action == "list":
        paginator = ec2.get_paginator("describe_instances")
        instances = []

        for page in paginator.paginate():
            for reservation in page["Reservations"]:
                for instance in reservation["Instances"]:
                    instances.append({
                        "id": instance["InstanceId"],
                        "state": instance["State"]["Name"],
                        "type": instance["InstanceType"]
                    })

        return {"instances": instances}

    instance_id = event.get("instance_id")
    if not instance_id:
        raise ValueError("instance_id is required")

    if action == "start":
        response = ec2.start_instances(InstanceIds=[instance_id])
    elif action == "stop":
        response = ec2.stop_instances(InstanceIds=[instance_id])
    else:
        raise ValueError("action must be list, start, or stop")

    return {"action": action, "response": response}
```

## Test events

List instances:

```json
{"action": "list"}
```

Start an instance:

```json
{"action": "start", "instance_id": "i-0123456789abcdef0"}
```

Stop an instance:

```json
{"action": "stop", "instance_id": "i-0123456789abcdef0"}
```

## Automation with EventBridge

You can use an EventBridge schedule to invoke the Lambda function at a chosen time. Use separate, explicit schedules or a controlled configuration to avoid stopping production instances accidentally.

## Production safety

- Restrict instance ARNs and optionally use resource tags with IAM conditions where supported.
- Validate the requested instance ID against an allowlist or approved tags.
- Exclude production and critical instances unless explicitly approved.
- Add dry-run or approval controls for destructive actions.
- Log the action, instance ID, and request ID without logging sensitive data.
