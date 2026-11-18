# AWS Lambda Automation for DevOps Engineers

AWS Lambda can automate common AWS operations without managing servers. With Python, Boto3, EventBridge, IAM, and CloudWatch, DevOps teams can build event-driven and scheduled automation.

## Folder contents

| File | What you will learn |
|---|---|
| [EC2 Auto Start/Stop](EC2-Auto-Start-Stop.md) | Start and stop EC2 instances on a schedule |
| [EBS Snapshot Automation](EBS-Snapshot-Automation.md) | Create and retain EBS snapshots |
| [S3 Lifecycle Automation](S3-Lifecycle-Automation.md) | Move or expire objects using lifecycle rules |
| [ECR Image Cleanup](ECR-Image-Cleanup.md) | Remove old, untagged container images safely |
| [Unused Resource Cleanup](Unused-Resource-Cleanup.md) | Find resources that may no longer be needed |
| [AWS Backup Automation](AWS-Backup-Automation.md) | Schedule and monitor AWS Backup jobs |
| [IAM Access Review](IAM-Access-Review.md) | Review IAM users, keys, and permissions |
| [Cost Optimization](Cost-Optimization.md) | Identify savings opportunities and automate safe actions |

## Basic automation architecture

```text
EventBridge schedule or AWS event
               |
               v
         Lambda function
               |
               v
         Python + Boto3
               |
               v
       AWS service API
               |
               v
       CloudWatch Logs
```

## Common DevOps automation tasks

- Start and stop non-production EC2 instances.
- Create EBS snapshots before maintenance.
- Apply S3 storage lifecycle policies.
- Clean up old container images in ECR.
- Identify unused or idle cloud resources.
- Start and monitor backup jobs.
- Review IAM access and credential age.
- Find cost optimization opportunities.

## Important safety rules

Automation can delete data, stop workloads, or change access. Use these practices:

1. Start with read-only discovery and a dry-run mode.
2. Scope actions to explicitly approved resource IDs, tags, accounts, and regions.
3. Exclude production resources unless there is a reviewed change plan.
4. Use least-privilege IAM permissions.
5. Log every proposed and completed action.
6. Add alerts and failure handling.
7. Test in a non-production account before production.
8. Require approval for destructive actions.
9. Make automation idempotent where possible.
10. Keep retention, recovery, and rollback requirements clear.

## Recommended learning path

1. Learn Python and Boto3 basics.
2. Create a Lambda function and execution role.
3. Trigger the function using EventBridge.
4. Add CloudWatch logging and alarms.
5. Implement read-only discovery.
6. Add dry-run output and approval controls.
7. Enable narrowly scoped actions only after testing.

## Official documentation

- [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Boto3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [Amazon EventBridge Scheduler](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)
- [AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [AWS Cost Optimization](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
