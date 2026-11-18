# AWS Lambda Environment Variables

## 1. Introduction

Environment variables are key-value pairs used to provide configuration to a Lambda function.

They allow you to change application settings without changing the source code.

Example:

```text
ENVIRONMENT = dev
APP_NAME    = my-lambda-app
LOG_LEVEL   = INFO
```

## 2. Why use environment variables?

- Separate code from configuration.
- Support development, testing, and production environments.
- Change application settings without editing the function code.
- Avoid hardcoding non-secret configuration values.

Do not treat ordinary environment variables as a secure secret store.

## 3. Architecture

```text
Lambda Configuration
        |
        v
Environment variables
        |
        v
  Lambda Function
        |
        v
Python reads values
        |
        v
Application behavior
```

## 4. Configure variables in AWS Console

1. Open AWS Lambda.
2. Select your function.
3. Open **Configuration**.
4. Select **Environment variables**.
5. Choose **Edit**.
6. Add key-value pairs.
7. Save the changes.

## 5. Configure variables using AWS CLI

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --environment "Variables={ENVIRONMENT=dev,APP_NAME=my-app,LOG_LEVEL=INFO}"
```

This replaces the function's environment variable map. Include every variable you want to retain.

## 6. Read variables using Python

```python
import os

environment = os.environ.get("ENVIRONMENT", "dev")
app_name = os.environ.get("APP_NAME", "default-app")
log_level = os.environ.get("LOG_LEVEL", "INFO")

def lambda_handler(event, context):
    print("Environment:", environment)
    print("Application:", app_name)
    print("Log level:", log_level)

    return {
        "environment": environment,
        "application": app_name
    }
```

## 7. Verify environment variables

```bash
aws lambda get-function-configuration \
  --function-name my-lambda-function \
  --query 'Environment.Variables'
```

Be careful when sharing command output because environment variables may contain sensitive values.

## 8. Environment variables using Terraform

Example:

```hcl
resource "aws_lambda_function" "demo" {
  function_name = "my-lambda-function"
  role          = aws_iam_role.lambda_role.arn
  runtime       = "python3.12"
  handler       = "lambda_function.lambda_handler"
  filename      = "function.zip"

  environment {
    variables = {
      ENVIRONMENT = "dev"
      APP_NAME    = "my-app"
      LOG_LEVEL   = "INFO"
    }
  }
}
```

This is a resource snippet. Use the complete role, package, provider, and deployment configuration from your Terraform project.

## 9. Secrets management

Do not store passwords, API tokens, or database credentials as plain-text environment variables in source code or IaC files.

Use:

- AWS Secrets Manager.
- AWS Systems Manager Parameter Store.
- Appropriate KMS encryption.
- IAM permissions that allow access only to required secrets.

Example: read a secret from Secrets Manager using Boto3.

```python
import boto3
import json

secrets = boto3.client("secretsmanager")

def lambda_handler(event, context):
    response = secrets.get_secret_value(
        SecretId="my-application-secret"
    )

    secret_value = json.loads(response["SecretString"])

    # Use the secret securely.
    # Do not print it to logs.

    return {"message": "Secret retrieved successfully"}
```

The Lambda execution role needs permission to retrieve the specific secret. Add KMS decrypt permission when required by the encryption setup.

## 10. Important limits and considerations

- Lambda environment variables have a total size quota.
- Environment variable values are strings.
- Configuration updates replace the configured variable map.
- Sensitive data should be managed through a dedicated secrets service.
- Changes to function configuration may briefly affect updates or invocation readiness.

Check current limits in the [AWS Lambda quotas documentation](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html).

## 11. Best practices

- Use clear and consistent variable names.
- Keep separate values for each environment.
- Do not hardcode secrets.
- Avoid logging sensitive configuration.
- Use Secrets Manager or Parameter Store for secrets.
- Manage configuration through IaC.
- Review IAM permissions for secret access.

## 12. Interview questions

**Q1. What are Lambda environment variables?**

They are key-value configuration settings available to function code at runtime.

**Q2. Why do we use environment variables?**

To keep configuration separate from application code.

**Q3. Should we store passwords in environment variables?**

Prefer Secrets Manager or Parameter Store for sensitive credentials and use IAM to control access.

**Q4. How do you read an environment variable in Python?**

Use `os.environ.get("VARIABLE_NAME")`.

## Summary

Environment variables make Lambda configuration flexible. Use them for ordinary settings and a dedicated secrets service for sensitive information.
