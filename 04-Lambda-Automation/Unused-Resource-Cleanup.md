# Unused AWS Resource Cleanup

## 1. Introduction

AWS accounts can contain resources that are no longer needed.

Examples include unattached EBS volumes, unused Elastic IP addresses, old snapshots, and abandoned development resources.

Lambda can help discover these resources and generate cleanup reports.

**Important:** A resource that looks unused may still be required for recovery, compliance, or a planned deployment.

## 2. Architecture

```text
EventBridge Scheduler
        |
        v
   Lambda Function
        |
        v
   Python + Boto3
        |
        v
  Discover AWS resources
        |
        v
 Generate review report
        |
        v
 SNS / CloudWatch Logs
```

## 3. Examples of resources to review

| Resource | Possible cleanup candidate |
|---|---|
| EBS volume | Unattached volume |
| Elastic IP | Unassociated address |
| ECR image | Old image outside retention requirements |
| EBS snapshot | Snapshot outside approved retention |
| EC2 instance | Stopped instance no longer required |
| Load balancer | Load balancer with no active workload |

Discovery does not prove that a resource is safe to delete.

## 4. Example: Find unattached EBS volumes

This read-only example lists volumes whose state is `available`.

```python
import boto3

ec2 = boto3.client("ec2")

def lambda_handler(event, context):
    response = ec2.describe_volumes(
        Filters=[
            {
                "Name": "status",
                "Values": ["available"]
            }
        ]
    )

    for volume in response.get("Volumes", []):
        print({
            "VolumeId": volume["VolumeId"],
            "Size": volume["Size"],
            "Tags": volume.get("Tags", [])
        })

    return {"message": "Discovery completed"}
```

An available volume is unattached, but it may contain important data. Do not automatically delete it based only on this status.

## 5. Example: Find unassociated Elastic IP addresses

```python
import boto3

ec2 = boto3.client("ec2")

def lambda_handler(event, context):
    response = ec2.describe_addresses()

    for address in response.get("Addresses", []):
        if "AssociationId" not in address:
            print("Unassociated Elastic IP:", address.get("PublicIp"))

    return {"message": "Elastic IP review completed"}
```

Review network and recovery requirements before releasing an address.

## 6. Recommended cleanup workflow

1. Discover candidate resources.
2. Collect IDs, tags, owners, age, and region.
3. Generate a report.
4. Send the report to resource owners.
5. Confirm retention and recovery requirements.
6. Obtain approval.
7. Take a backup if required.
8. Perform the approved action.
9. Record the result.

## 7. Security and operational controls

- Start with read-only IAM permissions.
- Use a dry-run mode.
- Use tag-based ownership and exclusions.
- Exclude production resources by default.
- Require approval before deletion.
- Keep an audit record.
- Limit the automation to approved accounts and regions.
- Add alerts for failures.

## 8. Interview questions

**Q1. How can Lambda help with unused AWS resources?**

It can discover resources using Boto3, generate reports, and support approved cleanup workflows.

**Q2. Should we automatically delete every unattached EBS volume?**

No. It may contain important data. Confirm ownership, age, backup, and retention requirements first.

**Q3. How do you make cleanup automation safe?**

Use read-only discovery, dry-run reports, allowlists, approval, least privilege, and audit logging.

## Summary

Build discovery and reporting first. Enable destructive cleanup only after clear ownership, retention, and approval controls are in place.
