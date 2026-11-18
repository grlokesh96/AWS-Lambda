# AWS Secrets Manager Integration with Lambda

AWS Secrets Manager stores sensitive values such as database credentials, API tokens, and third-party service keys. A Lambda function can retrieve a secret at runtime using its execution role.

## Architecture

```text
Lambda Function
      |
      | GetSecretValue
      v
AWS Secrets Manager
      |
      v
Encrypted Secret (AWS KMS)
```

## 1. Create a secret

```bash
aws secretsmanager create-secret \
  --name production/app/database \
  --description "Application database credentials" \
  --secret-string '{"username":"appuser","password":"replace-me"}' \
  --region ap-south-1
```

For real credentials, avoid placing the secret value directly in shell history or scripts. Use a secure creation workflow.

## 2. Grant the Lambda execution role access

Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:production/app/database-*"
  }]
}
```

If the secret uses a customer-managed KMS key, the role may also need `kms:Decrypt`, and the KMS key policy must allow the access.

## 3. Retrieve the secret using Python

Install the AWS SDK in your development environment if it is not already available:

```bash
pip install boto3
```

Example handler:

```python
import json
import os
import boto3

secrets = boto3.client("secretsmanager")

def lambda_handler(event, context):
    secret_arn = os.environ["DB_SECRET_ARN"]

    response = secrets.get_secret_value(SecretId=secret_arn)
    credentials = json.loads(response["SecretString"])

    # Use credentials to connect to the database.
    # Do not print or return the secret.
    return {"statusCode": 200, "body": "Secret retrieved"}
```

Set `DB_SECRET_ARN` to the secret ARN. The ARN is not the secret value, but it should still be treated as configuration information.

## 4. Test

```bash
aws secretsmanager get-secret-value \
  --secret-id production/app/database \
  --query ARN \
  --output text
```

This command returns the ARN only. Avoid printing secret values in terminals or CI logs.

## Caching

Fetching a secret on every invocation adds latency and API calls. For production workloads, consider the AWS Secrets Manager caching component or a carefully designed in-memory cache. Account for rotation: cached values can become stale, so refresh and retry safely when credentials change.

## Security best practices

- Never hardcode secrets in source code, Docker images, or Git repositories.
- Grant access to only the specific secret ARN.
- Use rotation where supported and test application behavior during rotation.
- Use a customer-managed KMS key when required by your security model.
- Avoid logging secret values or including them in error messages.
- Use a VPC endpoint if private network access to Secrets Manager is required.
