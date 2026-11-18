# IAM Least Privilege for AWS Lambda

IAM least privilege means granting a Lambda function only the permissions it needs to perform its job.

## Two important IAM policy types

1. **Execution role (identity-based policy):** Defines what the function can access, such as S3, DynamoDB, SQS, or Secrets Manager.
2. **Resource-based policy:** Defines who or what can invoke the function. See [Lambda-Resource-Policies.md](Lambda-Resource-Policies.md).

## Example: Lambda reads from one S3 bucket

Create an execution role with a trust policy that allows Lambda to assume it.

Trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "lambda.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

Example permissions policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "ReadOnlyFromOneBucket",
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::my-app-data/*"
  }]
}
```

If the function also needs to list the bucket, grant `s3:ListBucket` on the bucket ARN separately. Add only the actions actually required.

## Attach a policy

```bash
aws iam put-role-policy \
  --role-name lambda-s3-reader-role \
  --policy-name ReadOneBucket \
  --policy-document file://s3-read-policy.json
```

## Create a Lambda function with the role

```bash
aws lambda create-function \
  --function-name s3-reader \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --role arn:aws:iam::123456789012:role/lambda-s3-reader-role \
  --zip-file fileb://function.zip
```

The runtime and architecture must match the package you built and the runtimes currently supported by Lambda.

## Common execution-role permissions

| Use case | Example permissions |
|---|---|
| Write logs | `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` |
| Read S3 object | `s3:GetObject` |
| Write S3 object | `s3:PutObject` |
| Read secret | `secretsmanager:GetSecretValue` |
| Read DynamoDB item | `dynamodb:GetItem` |
| Send SQS message | `sqs:SendMessage` |
| Read SQS event source | SQS receive/delete and queue-attribute permissions as required |

For standard Lambda logging, the AWS-managed `AWSLambdaBasicExecutionRole` policy is commonly used. Review its permissions and scope custom policies where appropriate.

## Best practices

- Use separate roles for separate functions or workloads.
- Avoid `Action: "*"` and `Resource: "*"`, except where a service genuinely requires broader scope.
- Use IAM condition keys where they are supported and useful.
- Restrict `iam:PassRole` in deployment pipelines to approved roles.
- Review access with IAM Access Analyzer and CloudTrail.
- Do not confuse the Lambda execution role with the human or CI/CD deployment role.

## Troubleshooting

- `AccessDenied`: inspect the execution role, resource policy, permission boundaries, SCPs, and KMS key policy.
- S3 access denied: check bucket policy, object ARN, encryption key permissions, and cross-account access.
- Secret access denied: confirm `secretsmanager:GetSecretValue` and `kms:Decrypt` if a customer-managed key is used.
