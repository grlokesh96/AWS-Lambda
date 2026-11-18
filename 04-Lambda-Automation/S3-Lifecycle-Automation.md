# S3 Lifecycle Automation

## 1. Introduction

Amazon S3 Lifecycle rules automatically manage object storage over time.

They can transition objects to lower-cost storage classes or expire objects according to configured rules.

For most storage lifecycle requirements, use native S3 Lifecycle configuration rather than writing a Lambda function to move or delete objects.

## 2. Architecture

```text
Application uploads objects
          |
          v
       S3 Bucket
          |
          v
   S3 Lifecycle Rule
          |
       +--+--+
       |     |
       v     v
  Transition Expiration
  storage    eligible objects
       |     |
       v     v
 Lower-cost Object expiry
 storage
```

## 3. Common use cases

- Move older data to lower-cost storage classes.
- Expire temporary files.
- Remove incomplete multipart uploads after a configured period.
- Manage log retention.
- Keep storage costs under control.

## 4. Lifecycle rule example

The following JSON creates a lifecycle rule that transitions eligible objects under the `logs/` prefix to Standard-IA after 30 days and expires them after 365 days.

Check minimum storage duration, retrieval costs, object size considerations, and business retention requirements before applying a rule.

Create `lifecycle.json`:

```json
{
  "Rules": [
    {
      "ID": "logs-retention",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "logs/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
```

## 5. Apply the lifecycle configuration

```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket YOUR-BUCKET-NAME \
  --lifecycle-configuration file://lifecycle.json
```

This command replaces the bucket's existing lifecycle configuration. Review and merge existing rules before applying it.

## 6. View lifecycle configuration

```bash
aws s3api get-bucket-lifecycle-configuration \
  --bucket YOUR-BUCKET-NAME
```

## 7. Where does Lambda fit?

Lambda is useful when lifecycle decisions need custom logic, such as:

- Checking application metadata.
- Sending an approval notification.
- Creating a report about objects approaching expiration.
- Applying a custom workflow based on business rules.

For normal transitions and expiration, prefer S3 Lifecycle because AWS manages the process.

## 8. Security and safety

- Confirm the bucket and prefix.
- Review existing lifecycle rules before changes.
- Do not expire compliance or legally required data.
- Enable versioning where appropriate.
- Understand how lifecycle rules affect noncurrent object versions.
- Test with a non-production bucket.

## 9. Interview questions

**Q1. What is an S3 Lifecycle rule?**

It automatically transitions or expires objects based on configured rules.

**Q2. Do we always need Lambda for S3 lifecycle automation?**

No. S3 provides native lifecycle management. Lambda is useful only when custom logic or workflows are needed.

**Q3. How can lifecycle rules reduce costs?**

They can move eligible objects to lower-cost storage classes and expire data that is no longer required.

## Summary

Use native S3 Lifecycle rules for standard storage transitions and expiration. Use Lambda only when custom processing is required.
