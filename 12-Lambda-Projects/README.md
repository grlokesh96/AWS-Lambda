# AWS Lambda Projects

Hands-on AWS Lambda projects for DevOps, Cloud Automation, DevSecOps, and AIOps practice.

## Projects

| # | Project | Key services |
|---|---|---|
| 01 | [EC2 Auto Start/Stop](01-ec2-auto-start-stop.md) | Lambda, EventBridge, EC2, IAM |
| 02 | [Automated EBS Snapshots](02-automated-ebs-snapshots.md) | Lambda, EventBridge, EBS, DLM |
| 03 | [S3 File Processing](03-s3-file-processing.md) | S3, Lambda, IAM, CloudWatch |
| 04 | [Automated ECR Cleanup](04-automated-ecr-cleanup.md) | ECR, Lambda, EventBridge |
| 05 | [AWS Cost Optimization Agent](05-aws-cost-optimization-agent.md) | Lambda, Cost Explorer, SNS, Bedrock (optional) |
| 06 | [Automated Security Remediation](06-automated-security-remediation.md) | EventBridge, Lambda, Config/Security Hub, IAM |
| 07 | [Serverless CI/CD Pipeline](07-serverless-cicd-pipeline.md) | GitHub Actions, OIDC, Lambda, SAM |
| 08 | [Event-Driven Monitoring](08-event-driven-monitoring.md) | EventBridge, CloudWatch, Lambda, SNS |

## Recommended learning sequence

1. Start with EC2 scheduling and EBS snapshots.
2. Build event-driven S3 processing and ECR cleanup.
3. Add cost visibility and security remediation with approval controls.
4. Finish with CI/CD and event-driven monitoring.

## Repository setup

```bash
mkdir -p 12-lambda-projects
cd 12-lambda-projects
```

Use a dedicated IAM role for each Lambda function. Replace sample resource names, account IDs, ARNs, and regions with your own values. Test in a non-production AWS account first.

## Important safety notes

- Apply least-privilege IAM permissions; do not use `AdministratorAccess` for Lambda execution roles.
- Add tags to resources managed by automation and scope actions to those tags where possible.
- Use dry-run/report-only modes before enabling destructive actions.
- Configure CloudWatch alarms, retries, and a dead-letter destination where appropriate.
- Protect secrets with Secrets Manager or Parameter Store rather than embedding them in source code.
- Set budgets and review AWS service pricing before deploying resources.
