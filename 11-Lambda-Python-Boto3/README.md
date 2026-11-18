# AWS Lambda with Python and Boto3

This folder introduces Python for Lambda and practical AWS automation using Boto3, the AWS SDK for Python.

## Contents

| File | What you will learn |
|---|---|
| [Python-Lambda-Basics.md](Python-Lambda-Basics.md) | Lambda handler structure, events, context, and Python patterns |
| [Boto3-Automation.md](Boto3-Automation.md) | Use Boto3 clients and resources securely |
| [EC2-Automation-Script.md](EC2-Automation-Script.md) | Start, stop, and inspect EC2 instances |
| [S3-Automation-Script.md](S3-Automation-Script.md) | List, upload, and inspect S3 objects |
| [EKS-Automation-Script.md](EKS-Automation-Script.md) | Describe EKS clusters and automate supported operations |
| [CloudWatch-Automation.md](CloudWatch-Automation.md) | Read metrics and automate monitoring tasks |

## Architecture

```text
Event Source
(API Gateway / EventBridge / S3 / SQS)
              |
              v
        AWS Lambda
          Python
              |
              v
           Boto3
              |
              v
        AWS Services
  EC2 / S3 / EKS / CloudWatch
```

## Recommended project structure

```text
lambda-python-boto3/
├── src/
│   └── lambda_function.py
├── tests/
│   └── test_lambda_function.py
├── requirements.txt
└── README.md
```

## Prerequisites

- Python version supported by your Lambda runtime.
- AWS account and an appropriate Lambda execution role.
- AWS CLI for local testing and deployment.
- Boto3, which is available in supported Lambda Python runtimes; package and pin it yourself when you need a specific SDK version.

## Security practices

- Use the Lambda execution role for AWS credentials. Do not hardcode access keys.
- Grant only the required actions on the required resources.
- Validate event input and handle exceptions.
- Use pagination for list operations that can return multiple pages.
- Add timeouts and retries for external network calls.
- Do not log secrets or sensitive event payloads.
- Test automation against non-production resources first.
