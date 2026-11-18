# Project 04: Automated ECR Cleanup

## Objective

Remove old, untagged, or obsolete container images from Amazon Elastic Container Registry (ECR) to control storage costs.

## Architecture

```text
EventBridge schedule
        |
        v
 Lambda cleanup job
        |
        v
List ECR repositories/images
        |
        v
Apply retention policy
        |
        v
Batch delete eligible images
```

## Prefer ECR lifecycle policies

For common cleanup needs, use native ECR lifecycle policies. They are simpler and reduce custom code. Lambda is useful when retention rules depend on external release metadata, multiple accounts, approvals, or custom reporting.

## Example cleanup Lambda

The following sample targets **untagged images only** and keeps a configurable number of recent images. Test in a non-production repository first.

```python
import os
import boto3

ecr = boto3.client("ecr")
KEEP_COUNT = int(os.environ.get("KEEP_COUNT", "10"))
REPOSITORIES = os.environ.get("REPOSITORIES", "").split(",")

def lambda_handler(event, context):
    dry_run = event.get("dry_run", True)
    report = []

    for repository in [r.strip() for r in REPOSITORIES if r.strip()]:
        images = []
        paginator = ecr.get_paginator("describe_images")

        for page in paginator.paginate(repositoryName=repository):
            images.extend(page.get("imageDetails", []))

        untagged = [
            image for image in images
            if not image.get("imageTags")
        ]
        untagged.sort(
            key=lambda item: item.get("imagePushedAt"),
            reverse=True
        )

        # Keep the newest KEEP_COUNT untagged images; candidates are older.
        candidates = untagged[KEEP_COUNT:]
        identifiers = [
            {"imageDigest": item["imageDigest"]}
            for item in candidates if item.get("imageDigest")
        ]

        if dry_run or not identifiers:
            report.append({
                "repository": repository,
                "eligible": len(identifiers),
                "deleted": 0,
                "dry_run": dry_run
            })
            continue

        deleted = 0
        for offset in range(0, len(identifiers), 100):
            batch = identifiers[offset:offset + 100]
            result = ecr.batch_delete_image(
                repositoryName=repository,
                imageIds=batch
            )
            deleted += len(result.get("imageIds", []))

        report.append({
            "repository": repository,
            "eligible": len(identifiers),
            "deleted": deleted,
            "dry_run": False
        })

    return {"repositories": report}
```

This is a teaching example, not a universal retention policy. It does not account for deployment references, release channels, rollback windows, or cross-region replication. Consider keeping images by semantic version, release tag, or deployment metadata instead of deleting based only on age.

## Safe rollout

1. Start with `dry_run=true`.
2. Review the candidate list in logs or a report.
3. Confirm the retention policy with application owners.
4. Scope the Lambda role to named repositories.
5. Enable deletion only after approval.
6. Alert on API errors and unexpected deletion counts.

## IAM permissions

Typical actions: `ecr:DescribeRepositories`, `ecr:DescribeImages`, and `ecr:BatchDeleteImage`. Restrict repository-level permissions to approved repository ARNs.

## Best practices

- Use lifecycle policies where possible.
- Never delete `prod`, `stable`, or currently deployed images without a verified rollback strategy.
- Keep an audit record of repository, digest, tags, timestamp, and execution result.
- Remember that deleting an image manifest does not necessarily mean storage is reclaimed immediately in every scenario.
