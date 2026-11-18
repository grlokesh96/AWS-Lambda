# AWS Lambda Setup

This folder explains how to prepare your system and AWS account to create, deploy, and test an AWS Lambda function.

The examples use Ubuntu, AWS CLI, Python, and the AWS Management Console where useful.

## Folder contents

| File | What you will learn |
|---|---|
| [Install AWS CLI](Install-AWS-CLI.md) | Install and verify AWS CLI on Ubuntu |
| [Configure IAM Role](Configure-IAM-Role.md) | Create a Lambda execution role |
| [Create Lambda Function](Create-Lambda-Function.md) | Create a simple Python Lambda function |
| [Deploy Lambda Function](Deploy-Lambda-Function.md) | Package and deploy function code |
| [Test Lambda Function](Test-Lambda-Function.md) | Invoke the function and verify its output |

## Prerequisites

- An AWS account with permission to create IAM roles and Lambda functions.
- An Ubuntu machine or VM.
- Internet connectivity.
- Python 3 and `zip`.
- AWS CLI installed and configured.

Use an IAM identity with only the permissions required for this learning exercise. Do not use root account credentials for daily work.

## Setup flow

```text
Ubuntu VM
   |
   v
Install AWS CLI
   |
   v
Configure AWS credentials
   |
   v
Create Lambda execution role
   |
   v
Create Python function
   |
   v
Package and deploy code
   |
   v
Invoke and test function
   |
   v
Check CloudWatch Logs
```

## Quick verification

Check AWS CLI:

```bash
aws --version
```

Check which AWS identity is configured:

```bash
aws sts get-caller-identity
```

Check the default region:

```bash
aws configure get region
```

## Important security notes

- Prefer IAM Identity Center or another short-lived credential method for human access.
- Never commit AWS access keys, secret keys, session tokens, or `.env` files to GitHub.
- Use an IAM execution role for the Lambda function; do not put AWS credentials in the code.
- Choose a supported Python runtime. Runtime versions change, so check the [AWS Lambda runtimes documentation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html).
- Delete practice resources when you finish to avoid unnecessary charges.

## Official documentation

- [AWS CLI installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Lambda execution roles](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
