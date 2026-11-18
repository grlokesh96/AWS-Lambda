# AWS Lambda CI/CD

This folder explains how to build, test, package, deploy, and safely release AWS Lambda functions using common CI/CD platforms.

## Contents

| File | What you will learn |
|---|---|
| [Jenkins-Lambda-Pipeline.md](Jenkins-Lambda-Pipeline.md) | Build and deploy Lambda with Jenkins |
| [GitHub-Actions-Lambda.md](GitHub-Actions-Lambda.md) | Deploy Lambda using GitHub Actions and AWS OIDC |
| [GitLab-CI-Lambda.md](GitLab-CI-Lambda.md) | Build a Lambda pipeline with GitLab CI |
| [CodePipeline-Lambda.md](CodePipeline-Lambda.md) | AWS-native pipeline using CodePipeline and CodeBuild |
| [Lambda-Versioning.md](Lambda-Versioning.md) | Publish immutable Lambda versions |
| [Lambda-Aliases.md](Lambda-Aliases.md) | Use aliases to point to versions and control traffic |
| [Blue-Green-Deployment.md](Blue-Green-Deployment.md) | Release safely using traffic shifting |

## Typical CI/CD flow

```text
Developer
   |
   v
Git Repository
   |
   v
CI Pipeline
   |-- Lint and unit tests
   |-- Dependency / security checks
   |-- Package deployment artifact
   |
   v
AWS Authentication (OIDC or IAM role)
   |
   v
Update Lambda Function
   |
   v
Publish Version
   |
   v
Update Alias / Shift Traffic
   |
   v
Monitor and Roll Back if Needed
```

## Recommended practices

- Prefer short-lived credentials through OIDC or an assumed IAM role. Avoid long-lived AWS access keys in CI variables.
- Give the pipeline only the permissions required for its target function and deployment process.
- Run tests and security checks before deployment.
- Store deployment artifacts and retain build identifiers for traceability.
- Deploy to a test or staging environment before production.
- Publish versions and move aliases instead of sending production traffic to `$LATEST`.
- Use canary or linear traffic shifting for high-impact production releases.
- Monitor errors, duration, throttles, and application-specific metrics after deployment.

## Prerequisites

- An AWS account and a Lambda execution role.
- A Lambda function created in the target AWS Region.
- A repository containing the function source and tests.
- An IAM deployment role for the CI platform.
- AWS CLI and the relevant CI runner/build environment.

> Replace sample function names, account IDs, regions, role ARNs, and repository names with your own values. Verify current AWS service limits and platform syntax before using these examples in production.
