# Boto3 Automation with AWS Lambda

Boto3 is the AWS SDK for Python. It lets Lambda functions call AWS services such as EC2, S3, EKS, and CloudWatch.

## Boto3 client

A client maps closely to AWS service API operations:

```python
import boto3

ec2 = boto3.client("ec2", region_name="ap-south-1")

def lambda_handler(event, context):
    response = ec2.describe_instances()
    return {"reservations": len(response["Reservations"])}
```

## Boto3 resource

Resources provide a higher-level object-oriented interface for services that support them:

```python
import boto3

s3 = boto3.resource("s3")

def lambda_handler(event, context):
    bucket_names = [bucket.name for bucket in s3.buckets.all()]
    return {"buckets": bucket_names}
```

Not every AWS service has a Boto3 resource interface. Use a client when a resource interface is unavailable or when you need direct API operations.

## Authentication

Inside Lambda, Boto3 automatically uses the function's execution role credentials. Do not create or store long-lived access keys in code.

The execution role must allow the required API actions. For example:

- EC2 read: `ec2:DescribeInstances`
- S3 list: `s3:ListBucket`
- S3 upload: `s3:PutObject`
- CloudWatch metric read: `cloudwatch:GetMetricStatistics`
- EKS describe: `eks:DescribeCluster`

Scope resource-level permissions where the service supports resource scoping. Some `Describe` actions require `"Resource": "*"`.

## Pagination

Many AWS list APIs return paginated results. Use a paginator:

```python
import boto3

ec2 = boto3.client("ec2")

def get_all_instances():
    paginator = ec2.get_paginator("describe_instances")
    instances = []

    for page in paginator.paginate():
        for reservation in page["Reservations"]:
            instances.extend(reservation["Instances"])

    return instances
```

## Error handling

```python
import boto3
from botocore.exceptions import ClientError

s3 = boto3.client("s3")

def check_bucket(bucket_name):
    try:
        s3.head_bucket(Bucket=bucket_name)
        return True
    except ClientError:
        return False
```

For production, inspect the error code and handle expected failures separately rather than swallowing every error.

## Best practices

- Create SDK clients outside the handler to reuse them across warm invocations.
- Use the execution role, not static credentials.
- Use paginators for potentially large result sets.
- Add bounded retries and timeouts for operations where appropriate.
- Validate input and avoid logging secrets.
- Keep Boto3 and Botocore versions compatible when packaging them yourself.
