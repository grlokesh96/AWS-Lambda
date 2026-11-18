# AWS Backup Automation with Lambda

## 1. Introduction

AWS Backup is a managed service that centralizes and automates backups across supported AWS services.

You can use backup plans to define schedules, backup vaults, lifecycle settings, and resource assignments.

Lambda can support custom backup workflows, notifications, checks, and reporting.

For standard scheduled backups, prefer AWS Backup plans instead of writing custom snapshot code.

## 2. Architecture

```text
       AWS Backup Plan
              |
              v
      Scheduled backup
              |
              v
       Backup selection
              |
              v
      Supported resources
              |
              v
       Backup vault
              |
              v
       Recovery point

Lambda can monitor or report on the workflow.
```

## 3. Common use cases

- Schedule backups for supported resources.
- Monitor backup job status.
- Notify teams when a backup fails.
- Report recovery point age.
- Check backup coverage.
- Start an approved on-demand backup workflow.

## 4. Create a backup plan

Using the AWS Console:

1. Open AWS Backup.
2. Open **Backup plans**.
3. Choose **Create backup plan**.
4. Select a template or build a new plan.
5. Configure the backup schedule.
6. Select the backup vault.
7. Configure lifecycle and retention.
8. Assign resources using tags or resource selections.
9. Review and create the plan.

Check the AWS Backup documentation for service-specific support and configuration.

## 5. Start an on-demand backup using AWS CLI

Example for an EBS volume:

```bash
aws backup start-backup-job \
  --backup-vault-name YOUR-BACKUP-VAULT \
  --resource-arn YOUR_RESOURCE_ARN \
  --iam-role-arn YOUR_BACKUP_ROLE_ARN
```

Use the correct resource ARN and an IAM role that AWS Backup can assume. This command starts a backup job; it does not mean the backup has completed.

## 6. List backup jobs

```bash
aws backup list-backup-jobs \
  --by-state COMPLETED
```

You can also filter for other job states, such as `FAILED` or `RUNNING`.

## 7. Lambda monitoring example

A Lambda function can inspect recent backup jobs and log failed jobs.

```python
import boto3

backup = boto3.client("backup")

def lambda_handler(event, context):
    response = backup.list_backup_jobs(
        ByState="FAILED",
        MaxResults=50
    )

    for job in response.get("BackupJobs", []):
        print({
            "BackupJobId": job.get("BackupJobId"),
            "ResourceArn": job.get("ResourceArn"),
            "Status": job.get("State")
        })

    return {"message": "Backup review completed"}
```

For production, handle pagination and alerting, and define the time window you want to review.

## 8. Required permissions

Depending on the workflow, Lambda may need read permissions such as:

```text
backup:ListBackupJobs
backup:DescribeBackupJob
backup:ListRecoveryPointsByBackupVault
```

If Lambda starts backups, it needs additional permissions such as `backup:StartBackupJob`.

AWS Backup also needs its own service role to access and back up selected resources.

## 9. Important best practices

- Define recovery point objectives (RPO) and recovery time objectives (RTO).
- Configure retention based on business and compliance requirements.
- Protect backup vaults and recovery points.
- Monitor failed and incomplete backup jobs.
- Test restores regularly.
- Use AWS Backup policies where centralized multi-account management is required.
- Avoid deleting recovery points without approval.

## 10. Interview questions

**Q1. What is AWS Backup?**

It is a managed service for centralizing and automating backups across supported AWS services.

**Q2. Do we need Lambda to schedule backups?**

No. AWS Backup plans provide native scheduling. Lambda is useful for custom monitoring, reporting, and workflows.

**Q3. Why are restore tests important?**

A successful backup job does not guarantee that an application can be restored correctly. Restore tests verify recoverability.

## Summary

Use AWS Backup for standard backup policies. Use Lambda to add custom checks, notifications, and operational automation.
