# AWS Lambda Environment Variable Security

Environment variables are useful for non-secret configuration such as feature flags, service endpoints, and resource identifiers. They can also contain sensitive values, but a dedicated secret store is usually a better choice for credentials and API keys.

## Example configuration

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --environment 'Variables={APP_ENV=production,LOG_LEVEL=INFO,DB_SECRET_ARN=arn:aws:secretsmanager:ap-south-1:123456789012:secret:production/db-AbCdEf}' \
  --region ap-south-1
```

The example stores a secret ARN, not the database password itself.

## Read variables in Python

```python
import os

APP_ENV = os.environ.get("APP_ENV", "development")
LOG_LEVEL = os.environ.get("LOG_LEVEL", "INFO")
DB_SECRET_ARN = os.environ["DB_SECRET_ARN"]

def lambda_handler(event, context):
    return {"environment": APP_ENV}
```

## Security considerations

- Lambda encrypts environment variables at rest. You can configure a customer-managed KMS key for additional control.
- Users with permission to read function configuration may be able to view environment-variable values. Protect `lambda:GetFunctionConfiguration` and related access.
- Environment variables are available to the function process at runtime. Code execution within the function can read them.
- Avoid logging environment-variable values.
- Do not put credentials into source control, deployment manifests, or plain-text pipeline logs.

## Recommended pattern

```text
Lambda Environment
       |
       | Secret ARN / Secret Name
       v
Secrets Manager
       |
       | GetSecretValue (IAM-authorized)
       v
Runtime Secret Value
```

## Limit access to configuration

Use IAM to restrict who can update or read Lambda configuration. Scope permissions to the function ARN and separate developer, operator, and deployment roles.

## Best practices

- Use environment variables for configuration, not long-lived secrets.
- Store passwords and API tokens in Secrets Manager or Systems Manager Parameter Store with appropriate encryption.
- Use a dedicated execution role with access to only the required secret.
- Rotate secrets and handle refresh behavior.
- Review CI/CD logs and deployment artifacts for accidental secret exposure.
