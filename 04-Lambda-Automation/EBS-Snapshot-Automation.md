# EBS Snapshot Automation with AWS Lambda

## 1. Introduction

Amazon EBS snapshots are point-in-time backups of EBS volumes.

Lambda can automate snapshot creation. EventBridge can invoke the function on a schedule, for example before a maintenance window.

## 2. Architecture

```text
EventBridge Scheduler
        |
        v
   Lambda Function
        |
        | Boto3
        v
     EC2 / EBS API
        |
        v
 Create EBS snapshots
        |
        v
   CloudWatch Logs
```

## 3. How it works

1. EventBridge invokes Lambda.
2. Lambda finds approved EBS volumes.
3. Lambda creates snapshots.
4. The function adds identifying tags.
5. CloudWatch stores execution logs.
6. A separate retention process can remove expired snapshots after policy checks.

## 4. Example: Create a snapshot

```python
import boto3
from datetime import datetime, timezone

ec2 = boto3.client("ec2")

def lambda_handler(event, context):
    volume_id = "vol-REPLACE_ME"

    response = ec2.create_snapshot(
        VolumeId=volume_id,
        Description="Automated backup created by Lambda",
        TagSpecifications=[
            {
                "ResourceType": "snapshot",
                "Tags": [
                    {"Key": "CreatedBy", "Value": "Lambda"},
                    {"Key": "Purpose", "Value": "AutomatedBackup"},
                    {
                        "Key": "CreatedAt",
                        "Value": datetime.now(timezone.utc).isoformat()
                    }
                ]
            }
        ]
    )

    print("Snapshot ID:", response["SnapshotId"])
    return {"snapshot_id": response["SnapshotId"]}
```

Replace the volume ID with an approved EBS volume ID.

## 5. Required IAM permissions

A basic implementation may need:

```text
ec2:CreateSnapshot
ec2:DescribeVolumes
ec2:CreateTags
```

Add only the permissions required by your implementation. Restrict resources where the service supports resource-level permissions.

## 6. List snapshots

```bash
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[*].[SnapshotId,VolumeId,State,StartTime]' \
  --output table
```

## 7. Snapshot retention

Snapshots can accumulate over time and increase storage costs.

A retention process should:

1. Find snapshots created by the approved automation.
2. Check their creation time and retention tags.
3. Exclude snapshots required for recovery or compliance.
4. Produce a dry-run deletion report.
5. Delete only snapshots approved by the retention policy.

Example command to delete a specific snapshot:

```bash
aws ec2 delete-snapshot --snapshot-id snap-REPLACE_ME
```

Never run deletion against a broad list without validating ownership, tags, age, and recovery requirements.

## 8. Best practices

- Use AWS Backup when centralized backup policies and lifecycle management are needed.
- Tag snapshots with owner, application, and retention information.
- Monitor snapshot creation failures.
- Test restore procedures, not just snapshot creation.
- Consider application consistency requirements.
- Protect required recovery points from automated deletion.

## 9. Troubleshooting

| Problem | What to check |
|---|---|
| AccessDenied | IAM permissions |
| Invalid volume ID | Confirm the volume and region |
| Snapshot pending | Snapshot creation is still running |
| Snapshot cost increasing | Review retention and backup policy |
| Recovery fails | Test restore process and dependencies |

## 10. Interview questions

**Q1. What is an EBS snapshot?**

It is a point-in-time backup of an EBS volume.

**Q2. How can Lambda automate EBS snapshots?**

Lambda can use Boto3 `create_snapshot()` and be triggered by EventBridge Scheduler.

**Q3. Why are tags important?**

Tags help identify ownership, purpose, retention, and resources managed by automation.

## Summary

Lambda can automate EBS snapshot creation. Combine it with tagging, retention controls, monitoring, and regular restore tests.
