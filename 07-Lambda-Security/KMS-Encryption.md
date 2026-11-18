# AWS KMS Encryption for Lambda

AWS Key Management Service (KMS) creates and manages cryptographic keys used to protect data. Lambda integrates with KMS for supported encryption features, including environment-variable encryption with customer-managed keys.

## Encryption areas

| Data | Typical protection |
|---|---|
| Lambda environment variables | Encryption at rest; optionally use a customer-managed KMS key |
| Deployment ZIP artifacts | Encrypt the artifact in its storage location, such as an S3 bucket |
| Secrets Manager values | Encrypted by the service using KMS |
| Application data | Use service-side encryption or application-level encryption as required |
| Network traffic | TLS in transit |

## Create a customer-managed KMS key

```bash
aws kms create-key \
  --description "Lambda application configuration key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS \
  --region ap-south-1
```

Record the returned key ARN. Configure the key policy to permit only the intended administrators and workload roles.

## Configure Lambda to use a KMS key for environment variables

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --kms-key-arn arn:aws:kms:ap-south-1:123456789012:key/11111111-2222-3333-4444-555555555555 \
  --region ap-south-1
```

Lambda uses the configured key for supported environment-variable encryption operations. Verify the exact permissions required for your configuration and deployment workflow.

## KMS permissions

Depending on the feature and operation, relevant permissions can include:

- `kms:Encrypt`
- `kms:Decrypt`
- `kms:GenerateDataKey`
- `kms:DescribeKey`

Do not grant all of these automatically. Grant only the actions required by the service integration and caller. Both IAM policies and the KMS key policy can affect access.

## Inspect function configuration

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query '{FunctionName:FunctionName,KmsKeyArn:KMSKeyArn}'
```

## Best practices

- Prefer customer-managed keys when you need control over key policies, rotation settings, or audit requirements.
- Restrict key administration separately from key usage.
- Enable key rotation where appropriate for the key type and use case.
- Monitor key usage with CloudTrail.
- Plan key disablement and deletion carefully; disabling or deleting a key can make protected data unavailable.
- Do not treat encryption as a replacement for IAM authorization or secret management.

## Troubleshooting

- `AccessDeniedException`: check IAM permissions and the KMS key policy.
- Decryption failure: verify the correct key, encryption context, and caller permissions.
- Function update failure: verify the deployment principal can use the configured key as required.
