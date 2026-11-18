# Amazon ECR Image Cleanup Automation

## 1. Introduction

Amazon Elastic Container Registry (ECR) stores Docker and OCI container images.

Old and unused images can increase storage costs. ECR lifecycle policies can automatically expire images based on rules.

For standard cleanup, prefer ECR lifecycle policies. Lambda is useful for custom reporting, approval workflows, or cleanup across multiple repositories.

## 2. Architecture

```text
EventBridge Scheduler
        |
        v
   Lambda Function
        |
        v
      Boto3
        |
        v
  Amazon ECR API
        |
        v
List / review images
        |
        v
Report or approved cleanup
```

## 3. Common use cases

- Identify untagged images.
- Report old image versions.
- Review repository storage usage.
- Clean up images using an approved retention policy.
- Send cleanup reports to a DevOps team.

## 4. List ECR repositories

```bash
aws ecr describe-repositories \
  --query 'repositories[*].[repositoryName,repositoryUri]' \
  --output table
```

## 5. List images in a repository

```bash
aws ecr list-images \
  --repository-name YOUR-REPOSITORY \
  --filter tagStatus=UNTAGGED
```

## 6. Example: Report untagged images using Boto3

This example is read-only. It lists untagged images and does not delete anything.

```python
import boto3

ecr = boto3.client("ecr")

def lambda_handler(event, context):
    repository = "YOUR-REPOSITORY"

    paginator = ecr.get_paginator("list_images")

    for page in paginator.paginate(
        repositoryName=repository,
        filter={"tagStatus": "UNTAGGED"}
    ):
        for image in page.get("imageIds", []):
            print("Untagged image:", image)

    return {"message": "Image review completed"}
```

## 7. ECR lifecycle policy example

This example expires untagged images after 14 days.

Create `ecr-lifecycle-policy.json`:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Expire untagged images older than 14 days",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 14
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

Apply the policy:

```bash
aws ecr put-lifecycle-policy \
  --repository-name YOUR-REPOSITORY \
  --lifecycle-policy-text file://ecr-lifecycle-policy.json
```

This changes the repository's lifecycle policy. Review existing rules and combine them carefully before applying.

## 8. Important safety points

- Do not delete images currently used by running deployments.
- Preserve release and rollback images.
- Review image tags and digest references.
- Test lifecycle policies before production use.
- Use immutable release tags where suitable.
- Keep retention requirements documented.

## 9. Required permissions

For read-only reporting, consider:

```text
ecr:DescribeRepositories
ecr:ListImages
ecr:DescribeImages
```

Lifecycle policy management requires additional permissions, such as `ecr:PutLifecyclePolicy`.

## 10. Interview questions

**Q1. What is Amazon ECR?**

It is a managed registry for storing container images.

**Q2. How can old ECR images be cleaned up?**

Use ECR lifecycle policies for standard expiration rules. Lambda can support custom reporting and approval workflows.

**Q3. Why should we preserve some old images?**

They may be required for rollback, incident recovery, or compliance.

## Summary

ECR lifecycle policies are the preferred method for routine image retention. Use Lambda when additional automation or reporting is needed.
