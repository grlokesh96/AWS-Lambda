# AWS Lambda Security

This folder covers the main security controls for AWS Lambda, from IAM permissions and network access to secrets, encryption, code scanning, container images, and resource-based policies.

## Contents

| File | Topic |
|---|---|
| [IAM-Least-Privilege.md](IAM-Least-Privilege.md) | Restrict Lambda execution and deployment permissions |
| [Lambda-VPC-Security.md](Lambda-VPC-Security.md) | Secure network access for VPC-connected functions |
| [Secrets-Manager-Integration.md](Secrets-Manager-Integration.md) | Retrieve and protect application secrets |
| [KMS-Encryption.md](KMS-Encryption.md) | Encrypt Lambda data using AWS KMS |
| [Environment-Variable-Security.md](Environment-Variable-Security.md) | Protect configuration and environment variables |
| [Lambda-Code-Scanning.md](Lambda-Code-Scanning.md) | Scan source code and dependencies |
| [Lambda-Container-Image-Security.md](Lambda-Container-Image-Security.md) | Secure Lambda container images |
| [Lambda-Resource-Policies.md](Lambda-Resource-Policies.md) | Control who can invoke a Lambda function |

## Lambda security layers

```text
                 Users / AWS Services
                         |
                  Resource Policy
                         |
                         v
                  AWS Lambda Function
                   /      |       \
                  /       |        \
          Execution IAM  Network   Encryption
              Role       Controls   (KMS)
                  \       |        /
                   \      |       /
                 Secure Application
                         |
             Secrets + Code / Image Scanning
```

## Security checklist

- [ ] Use a dedicated execution role for each function or closely related workload.
- [ ] Grant only required actions on specific resources.
- [ ] Restrict who can invoke the function.
- [ ] Keep secrets out of source code and plain-text configuration.
- [ ] Encrypt sensitive data at rest and use TLS in transit.
- [ ] Place a function in a VPC only when it needs private network access.
- [ ] Scan source code, dependencies, and container images.
- [ ] Patch dependencies and rebuild images regularly.
- [ ] Enable appropriate CloudTrail, CloudWatch Logs, and security monitoring.
- [ ] Review permissions and resource policies periodically.

## Important distinction

A Lambda execution role controls what the function can access. A Lambda resource-based policy controls which principals or services can invoke the function. Both must be configured correctly.

> Replace sample ARNs, account IDs, function names, secret names, and regions with your own values. Validate policies in a non-production account before rollout.
