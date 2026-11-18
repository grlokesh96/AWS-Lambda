# Install AWS CLI on Ubuntu

## 1. Introduction

AWS CLI (Command Line Interface) lets you manage AWS services from a terminal.

As a DevOps Engineer, you can use AWS CLI to create resources, deploy applications, automate tasks, and troubleshoot AWS services.

## 2. Update Ubuntu packages

```bash
sudo apt update
sudo apt upgrade -y
```

## 3. Install required tools

```bash
sudo apt install -y curl unzip
```

## 4. Download AWS CLI v2

For a typical 64-bit x86 Ubuntu VM:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

If your machine uses ARM64, use the ARM64 installer from the [official AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

## 5. Extract the installer

```bash
unzip awscliv2.zip
```

## 6. Install AWS CLI

```bash
sudo ./aws/install
```

If AWS CLI is already installed and you want to update it, follow the update instructions in the official guide.

## 7. Verify installation

```bash
aws --version
```

Example output:

```text
aws-cli/2.x.x Python/3.x.x Linux/...
```

The exact version will depend on the release installed.

## 8. Configure AWS CLI

For a simple local learning setup:

```bash
aws configure
```

The command prompts for:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Use credentials from an appropriately scoped IAM identity. Never paste credentials into GitHub or share them in logs.

For longer-term human access, prefer AWS IAM Identity Center or another supported short-lived credential approach.

## 9. Verify AWS identity

```bash
aws sts get-caller-identity
```

This command displays the AWS account, caller ARN, and user or role ID for the active credentials.

## 10. Check configured region

```bash
aws configure get region
```

Example:

```text
ap-south-1
```

Choose the region where you plan to create your Lambda function.

## 11. Useful AWS CLI commands

```bash
aws --version
aws configure list
aws sts get-caller-identity
aws configure get region
```

## 12. Troubleshooting

### Error: `aws: command not found`

Check whether the installer completed successfully:

```bash
which aws
```

If necessary, open a new terminal and verify your PATH.

### Error: Unable to locate credentials

Configure an approved credential provider, such as IAM Identity Center, an instance role, or environment-specific credentials.

### Error: AccessDenied

The active IAM identity may not have permission to perform the requested operation. Ask an administrator to grant the minimum required permissions.

## Summary

AWS CLI is an important tool for AWS DevOps automation. Install it, configure a secure identity, and verify access before creating Lambda resources.
