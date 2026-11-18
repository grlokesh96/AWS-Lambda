# AWS Lambda Configuration

## 1. Introduction

Lambda configuration controls how a function runs.

You can configure memory, timeout, environment variables, IAM permissions, networking, architecture, and other settings.

## 2. Important configuration options

| Setting | Purpose |
|---|---|
| Memory | Memory allocated to the function; CPU allocation also increases with memory |
| Timeout | Maximum time allowed for one invocation |
| Environment variables | Configuration values available to the function |
| Execution role | IAM permissions used by the function |
| VPC | Allows the function to connect to resources in a VPC |
| Architecture | Instruction set architecture, such as x86_64 or arm64 |
| Ephemeral storage | Temporary `/tmp` storage available during execution |
| Concurrency | Controls how many invocations can run at the same time |

## 3. Memory

Memory is configured in MB.

More memory also provides more CPU power. Increasing memory may improve performance, but it can also increase the cost per unit of execution time.

Choose memory by testing the function with realistic workloads.

## 4. Timeout

Timeout defines the maximum execution duration for one invocation.

For example, if a function has a timeout of 30 seconds, Lambda stops the invocation when that limit is reached.

Example using AWS CLI:

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --timeout 30
```

## 5. Environment variables

Environment variables store configuration values that your code reads at runtime.

Example:

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --environment "Variables={ENV=dev,BUCKET_NAME=my-app-bucket}"
```

Python example:

```python
import os

environment = os.environ.get("ENV", "dev")
bucket_name = os.environ.get("BUCKET_NAME")

def lambda_handler(event, context):
    return {
        "environment": environment,
        "bucket": bucket_name
    }
```

Do not store plain-text passwords, access keys, or other secrets in environment variables. Use AWS Secrets Manager or Systems Manager Parameter Store, with suitable encryption and IAM permissions, for sensitive values.

## 6. IAM execution role

The execution role controls which AWS services the function can access.

For example, a function that reads files from S3 needs suitable S3 read permissions.

Basic flow:

```text
Lambda function
      |
      v
Assumes execution role
      |
      v
IAM policies checked
      |
      v
Access allowed or denied
```

Follow least privilege. Do not give every function administrator permissions.

## 7. VPC configuration

A Lambda function can be connected to a VPC when it needs access to private resources, such as an RDS database.

Important points:

- Select suitable subnets.
- Configure security groups.
- Ensure routing and network access are correct.
- Provide NAT or suitable VPC endpoints when outbound access is required.

Connecting Lambda to a VPC does not automatically provide internet access.

## 8. Useful AWS CLI commands

Get function configuration:

```bash
aws lambda get-function-configuration \
  --function-name my-lambda-function
```

Update timeout:

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --timeout 60
```

Update memory:

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --memory-size 512
```

Invoke a function:

```bash
aws lambda invoke \
  --function-name my-lambda-function \
  --payload '{}' \
  response.json
```

View the response:

```bash
cat response.json
```

## 9. DevOps best practices

- Keep development, testing, and production configuration separate.
- Use IAM roles with least privilege.
- Store secrets in a secrets-management service.
- Set realistic memory and timeout values.
- Use infrastructure as code to manage configuration consistently.
- Monitor duration, errors, and throttling.
- Test configuration changes before production rollout.

## 10. Interview questions

**Q1. What is Lambda timeout?**

It is the maximum duration allowed for a single function invocation.

**Q2. What are environment variables?**

They are configuration values that the function can read while running.

**Q3. Why does Lambda need an IAM role?**

The role provides permissions for the function to access AWS services.

**Q4. Does Lambda need a VPC?**

Not always. A VPC is needed when the function must access private resources in a VPC.

## Summary

Correct Lambda configuration improves security, performance, reliability, and cost control.
