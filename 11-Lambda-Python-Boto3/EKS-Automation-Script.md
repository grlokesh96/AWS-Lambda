# EKS Automation with Python and Boto3

Boto3 can manage and inspect Amazon EKS control-plane resources. It can describe clusters, list clusters, and retrieve cluster connection information.

> Important: Boto3's EKS client manages the EKS AWS API. It does not replace `kubectl` for Kubernetes API operations such as listing Pods or Deployments.

## IAM permissions

For read-only cluster discovery, the Lambda execution role may need:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "eks:ListClusters",
      "eks:DescribeCluster"
    ],
    "Resource": "*"
  }]
}
```

`ListClusters` generally requires `"Resource": "*"`. Scope `DescribeCluster` to approved cluster ARNs where possible.

## List EKS clusters

```python
import boto3

eks = boto3.client("eks", region_name="ap-south-1")

def lambda_handler(event, context):
    paginator = eks.get_paginator("list_clusters")
    clusters = []

    for page in paginator.paginate():
        clusters.extend(page.get("clusters", []))

    return {"clusters": clusters}
```

## Describe a cluster

```python
import boto3

eks = boto3.client("eks", region_name="ap-south-1")

def lambda_handler(event, context):
    cluster_name = event["cluster_name"]
    response = eks.describe_cluster(name=cluster_name)
    cluster = response["cluster"]

    return {
        "name": cluster["name"],
        "status": cluster["status"],
        "version": cluster["version"],
        "endpoint": cluster["endpoint"],
        "platform_version": cluster["platformVersion"]
    }
```

## Example event

```json
{
  "cluster_name": "dev-eks-cluster"
}
```

## Performing Kubernetes API operations

If Lambda must call the Kubernetes API, additional setup is required:

1. Provide network connectivity to the EKS cluster endpoint.
2. Generate authentication using an appropriate IAM identity and EKS token flow.
3. Grant the IAM principal access to the cluster through the EKS access configuration and Kubernetes authorization.
4. Package and configure a Kubernetes client library, if needed.
5. Apply least-privilege Kubernetes RBAC.

The Lambda execution role's `eks:DescribeCluster` permission alone does not grant Kubernetes API access.

## Best practices

- Use read-only automation for inventory and status checks where possible.
- Restrict cluster ARNs and Regions.
- Avoid exposing cluster endpoints or sensitive configuration in logs.
- Use private networking when the cluster endpoint is private.
- Add explicit approval for destructive cluster operations.
