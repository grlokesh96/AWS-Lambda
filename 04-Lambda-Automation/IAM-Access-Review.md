# IAM Access Review Automation

## 1. Introduction

AWS Identity and Access Management (IAM) controls who can access AWS resources and what actions they can perform.

Regular access reviews help identify unused credentials, excessive permissions, and accounts that need attention.

Lambda and Boto3 can automate read-only IAM reports.

## 2. Architecture

```text
EventBridge Scheduler
        |
        v
   Lambda Function
        |
        v
      Boto3 IAM
        |
        v
 Review users, keys,
 policies and roles
        |
        v
 Generate report
        |
        v
 CloudWatch / SNS
```

## 3. Common review tasks

- List IAM users.
- Review access key age and status.
- Identify users with console passwords.
- Review attached policies.
- Identify inactive credentials.
- Review roles and trust policies.
- Check MFA status for IAM users where applicable.

For human access, consider IAM Identity Center and federation rather than long-term IAM user credentials.

## 4. Example: List IAM users

```python
import boto3

iam = boto3.client("iam")

def lambda_handler(event, context):
    paginator = iam.get_paginator("list_users")

    for page in paginator.paginate():
        for user in page.get("Users", []):
            print({
                "UserName": user["UserName"],
                "CreateDate": user["CreateDate"].isoformat()
            })

    return {"message": "IAM user review completed"}
```

## 5. Example: Review access keys

```python
import boto3

iam = boto3.client("iam")

def lambda_handler(event, context):
    paginator = iam.get_paginator("list_users")

    for page in paginator.paginate():
        for user in page.get("Users", []):
            keys = iam.list_access_keys(
                UserName=user["UserName"]
            )

            for key in keys.get("AccessKeyMetadata", []):
                print({
                    "User": user["UserName"],
                    "AccessKeyId": key["AccessKeyId"],
                    "Status": key["Status"],
                    "Created": key["CreateDate"].isoformat()
                })

    return {"message": "Access key review completed"}
```

This example reports metadata only. Do not print secret access keys; AWS does not return their secret values in this listing operation.

## 6. Useful AWS CLI commands

List IAM users:

```bash
aws iam list-users
```

List access keys for a user:

```bash
aws iam list-access-keys \
  --user-name YOUR-IAM-USER
```

List attached user policies:

```bash
aws iam list-attached-user-policies \
  --user-name YOUR-IAM-USER
```

Get credential report:

```bash
aws iam generate-credential-report
```

Download the report after it is ready:

```bash
aws iam get-credential-report
```

## 7. Required IAM permissions

A read-only review function may need permissions such as:

```text
iam:ListUsers
iam:ListAccessKeys
iam:ListAttachedUserPolicies
iam:GenerateCredentialReport
iam:GetCredentialReport
```

Add only the actions needed by the report.

## 8. Best practices

- Begin with read-only reporting.
- Do not automatically delete users or access keys.
- Confirm ownership before changing credentials.
- Prefer temporary credentials and federation.
- Enforce MFA where appropriate.
- Review policies for excessive permissions.
- Use IAM Access Analyzer to identify external access and policy risks.
- Send reports only to approved recipients.

## 9. Interview questions

**Q1. What is IAM?**

IAM is an AWS service used to manage identities and permissions.

**Q2. Why review access keys?**

Old or unused keys increase the risk of unauthorized access if they are exposed or misused.

**Q3. How can Lambda automate IAM reviews?**

Lambda can use Boto3 to collect IAM metadata, identify review candidates, and generate reports.

**Q4. Should Lambda automatically delete inactive users?**

No. Confirm ownership, dependencies, and approval before making access changes.

## Summary

Lambda can automate IAM access reporting. Start with read-only reviews and require human approval for permission or credential changes.
