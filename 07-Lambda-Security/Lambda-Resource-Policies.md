# AWS Lambda Resource-Based Policies

A Lambda resource-based policy controls which AWS accounts, IAM principals, or AWS services can invoke a function. It complements the function's execution role.

## Execution role vs resource policy

| Policy | Controls |
|---|---|
| Execution role | What the function can access after it runs |
| Resource-based policy | Who or what can invoke the function |

For example, an S3 bucket notification invokes Lambda. The Lambda resource policy must allow the S3 service principal to invoke the function, scoped to the source bucket and account.

## Allow an S3 bucket to invoke Lambda

```bash
aws lambda add-permission \
  --function-name demo-lambda \
  --statement-id AllowS3Invoke \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-app-upload-bucket \
  --source-account 123456789012 \
  --region ap-south-1
```

The S3 bucket notification configuration must also be set up to target the function.

## Allow EventBridge to invoke Lambda

```bash
aws lambda add-permission \
  --function-name demo-lambda \
  --statement-id AllowEventBridgeInvoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:ap-south-1:123456789012:rule/my-schedule-rule \
  --region ap-south-1
```

Configure the EventBridge rule target separately.

## Inspect the policy

```bash
aws lambda get-policy \
  --function-name demo-lambda \
  --region ap-south-1
```

Remove a statement when it is no longer required:

```bash
aws lambda remove-permission \
  --function-name demo-lambda \
  --statement-id AllowS3Invoke \
  --region ap-south-1
```

## Cross-account invocation

For cross-account invocation, the function's resource policy must allow the external account or principal, and the caller's identity policy must also authorize `lambda:InvokeFunction`. Both sides of the authorization model matter.

## Security best practices

- Scope service permissions with `source-arn` and `source-account` where supported.
- Avoid wildcard principals unless there is a deliberate, reviewed use case.
- Use unique statement IDs so policy statements can be managed cleanly.
- Review policies regularly and remove obsolete permissions.
- Prefer invoking a qualified alias when your integration should target a controlled release.
- Check both identity policies and resource policies when troubleshooting access.

## Troubleshooting

- `AccessDenied`: check the caller's identity policy and the Lambda resource policy.
- Service cannot invoke Lambda: verify the service principal, source ARN, source account, Region, and target configuration.
- Permission already exists: use a unique statement ID or inspect the existing policy before changing it.
