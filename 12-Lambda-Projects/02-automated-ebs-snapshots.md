# Project 02: Automated EBS Snapshots

## Objective

Create scheduled snapshots of EBS volumes, apply retention, and protect recovery points from accidental deletion.

## Architecture

```text
EventBridge schedule
        |
        v
   Lambda (Boto3)
        |
        v
Describe tagged EBS volumes
        |
        v
Create snapshots + tags
        |
        v
CloudWatch Logs / SNS alert
```

## Recommended approach

For standard backup policies, evaluate **Amazon Data Lifecycle Manager (DLM)** first. DLM provides policy-based EBS snapshot lifecycle management without maintaining custom cleanup code. Use Lambda when you need custom workflows, cross-account coordination, notifications, or application-specific logic.

## Tagging strategy

Tag eligible volumes with:

```text
Backup=Daily
Environment=NonProd
```

Never assume every volume should be backed up. Exclude ephemeral or otherwise unsuitable volumes based on your workload policy.

## Lambda example

```python
import boto3
from datetime import datetime, timezone

ec2 = boto3.client("ec2")

def lambda_handler(event, context):
    paginator = ec2.get_paginator("describe_volumes")
    created = []

    for page in paginator.paginate(
        Filters=[{"Name": "tag:Backup", "Values": ["Daily"]}]
    ):
        for volume in page["Volumes"]:
            volume_id = volume["VolumeId"]
            response = ec2.create_snapshot(
                VolumeId=volume_id,
                Description=f"Scheduled backup for {volume_id}"
            )
            snapshot_id = response["SnapshotId"]

            ec2.create_tags(
                Resources=[snapshot_id],
                Tags=[
                    {"Key": "CreatedBy", "Value": "LambdaEbsBackup"},
                    {"Key": "SourceVolume", "Value": volume_id},
                    {"Key": "CreatedAt", "Value": datetime.now(timezone.utc).isoformat()}
                ]
            )
            created.append(snapshot_id)

    return {"snapshots_created": created, "count": len(created)}
```

This example creates snapshots but intentionally does not delete old snapshots. Add retention only after defining recovery requirements and validating ownership tags.

## IAM permissions

Typical permissions include `ec2:DescribeVolumes`, `ec2:CreateSnapshot`, and `ec2:CreateTags`. If implementing cleanup, add narrowly scoped `ec2:DescribeSnapshots` and `ec2:DeleteSnapshot`. Restrict destructive permissions by resource tags and approved account/region wherever supported.

## Deploy and schedule

1. Create the Lambda execution role.
2. Deploy the Python function.
3. Configure an EventBridge schedule (for example, daily at 02:00 UTC).
4. Grant EventBridge permission to invoke the function.
5. Send a test event and inspect CloudWatch Logs.

## Retention design

Define retention from recovery objectives, not an arbitrary number. Example policy: retain daily snapshots for 7 days and weekly snapshots for 4 weeks. Before deleting, verify:
- Snapshot has the expected `CreatedBy` tag.
- Snapshot belongs to the intended account and region.
- Snapshot is not part of a legal hold or application recovery plan.
- The minimum number of valid recovery points remains available.

## Operational best practices

- Use DLM for straightforward snapshot schedules.
- Consider application consistency; coordinate writes or use application-aware backup procedures where required.
- Monitor snapshot completion and failures, not just API acceptance.
- Enable encryption and use appropriate KMS key policies.
- Test restores regularly. A successful snapshot does not prove that an application can be recovered.
