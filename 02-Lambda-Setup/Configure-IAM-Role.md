# Configure IAM Role for AWS Lambda

## 1. Introduction

An IAM role gives an AWS Lambda function permission to access other AWS services.

For example, if a Lambda function needs to read an object from S3, its execution role must allow the required S3 read action on the relevant bucket.

**Important:** The Lambda execution role is different from the IAM identity you use to create or deploy the function.

## 2. Why does Lambda need an IAM role?

Example:

```text
Lambda function
      |
      v
Assumes IAM execution role
      |
      v
IAM policy checks permissions
      |
      v
Access AWS service
(e.g. S3 or CloudWatch Logs)
```

Without the required permissions, AWS API calls made by the function can fail with `AccessDenied`.

## 3. Create an execution role in the AWS Console

1. Open the AWS Management Console.
2. Search for **IAM**.
3. Select **Roles**.
4. Choose **Create role**.
5. Select **AWS service** as the trusted entity type.
6. Choose **Lambda** as the use case.
7. Attach the required permissions.
8. Give the role a name, for example `lambda-basic-execution-role`.
9. Create the role.

For basic logging, attach the AWS managed policy:

```text
AWSLambdaBasicExecutionRole
```

This policy allows the function to create log streams and write logs to CloudWatch Logs. Review the policy before using it in production.

## 4. Create a role using AWS CLI

### Step 1: Create the trust policy

Create a file named `trust-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This trust policy allows the Lambda service to assume the role.

### Step 2: Create the IAM role

```bash
aws iam create-role \
  --role-name lambda-basic-execution-role \
  --assume-role-policy-document file://trust-policy.json
```

### Step 3: Attach the basic logging policy

```bash
aws iam attach-role-policy \
  --role-name lambda-basic-execution-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

### Step 4: Get the role ARN

```bash
aws iam get-role \
  --role-name lambda-basic-execution-role \
  --query 'Role.Arn' \
  --output text
```

Save the returned ARN. You will use it when creating the Lambda function.

## 5. Add permissions for S3 (only if needed)

For a function that reads objects from one bucket, create a policy scoped to that bucket and its objects.

Example `s3-read-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

Replace `YOUR-BUCKET-NAME` with your bucket name.

Create and attach the policy:

```bash
aws iam put-role-policy \
  --role-name lambda-basic-execution-role \
  --policy-name LambdaReadSpecificBucket \
  --policy-document file://s3-read-policy.json
```

Do not add permissions that the function does not need.

## 6. Verify the role

```bash
aws iam get-role --role-name lambda-basic-execution-role
```

List attached managed policies:

```bash
aws iam list-attached-role-policies \
  --role-name lambda-basic-execution-role
```

## 7. Common IAM issues

| Error | Possible reason |
|---|---|
| AccessDenied | Required action is not allowed by the role's policies |
| Role cannot be assumed | Trust policy is incorrect |
| Logs not appearing | Logging permissions or log group configuration may be missing |
| S3 access denied | Bucket/object permissions or bucket policy may be blocking access |

## 8. Best practices

- Follow least privilege.
- Use a separate execution role for different workloads where appropriate.
- Restrict S3 permissions to required buckets and paths.
- Do not place access keys in Lambda code.
- Review IAM policies regularly.
- Use IAM Access Analyzer to help review access.

## 9. Interview questions

**Q1. Why does Lambda need an IAM role?**

The execution role gives Lambda permission to access AWS services while the function runs.

**Q2. What is a trust policy?**

It defines which principal is allowed to assume an IAM role. For Lambda, the trusted service principal is `lambda.amazonaws.com`.

**Q3. What is the difference between an execution role and a deployment identity?**

The execution role is assumed by Lambda at runtime. The deployment identity is used by a person or CI/CD system to create or update the function.

## Summary

Create an IAM execution role, allow the Lambda service to assume it, and attach only the permissions required by your function.
