# AWS Lambda Permission Denied Errors

Permission errors occur when a principal or Lambda execution role does not have the required authorization. The missing permission may be in an identity policy, resource policy, permissions boundary, service control policy, or KMS key policy.

## Common error messages

```text
AccessDeniedException
AccessDenied
not authorized to perform: ...
User is not authorized to perform: ...
```

## Understand the two main Lambda policy types

| Policy | Purpose |
|---|---|
| Execution role | Controls what the function can access |
| Resource-based policy | Controls who can invoke the function |

Also check the caller's IAM identity policy when a user, service, or application invokes Lambda.

## Step 1: Identify the denied action

Read the full error message. Identify:

- Principal or assumed role.
- Denied action, such as `s3:GetObject`.
- Resource ARN.
- AWS account and Region.

## Step 2: Inspect the execution role

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query Role
```

Inspect attached policies:

```bash
aws iam list-attached-role-policies \
  --role-name lambda-execution-role

aws iam list-role-policies \
  --role-name lambda-execution-role
```

## Step 3: Add a narrowly scoped permission

Example: allow reading objects from one bucket.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-app-data/*"
  }]
}
```

Do not use `Action: "*"` or `Resource: "*"` as a quick fix unless there is a carefully justified requirement.

## Step 4: Check resource policies

For S3, SQS, SNS, EventBridge, and cross-account resources, inspect the target resource policy. For Lambda invocation, inspect the Lambda resource policy:

```bash
aws lambda get-policy \
  --function-name demo-lambda
```

## Step 5: Check KMS permissions

If the resource is encrypted with a customer-managed KMS key, verify both IAM permissions and the key policy. The required permissions depend on the operation, such as `kms:Decrypt`.

## Other causes to check

- Permissions boundary denies the action.
- AWS Organizations SCP denies the action.
- Explicit `Deny` in any applicable policy.
- Incorrect resource ARN, account ID, or Region.
- Missing `iam:PassRole` in the deployment pipeline.
- Service principal or source ARN mismatch for trigger invocation.

## Best practices

- Use IAM Access Analyzer and CloudTrail to investigate access.
- Scope permissions to exact resources.
- Separate deployment permissions from runtime permissions.
- Test changes with the same role and invocation path used in production.
