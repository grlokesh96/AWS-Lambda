# AWS Lambda Fundamentals

AWS Lambda is a serverless service from AWS. It runs your code when an event happens, without requiring you to create or manage servers.

This folder covers the basic concepts every AWS DevOps Engineer should understand.

## What you will learn

| File | Topic |
|---|---|
| [What is AWS Lambda?](What-is-AWS-Lambda.md) | Lambda basics, benefits, and common uses |
| [Lambda Architecture](Lambda-Architecture.md) | How events reach and invoke a function |
| [Lambda Execution Model](Lambda-Execution-Model.md) | What happens when a function runs |
| [Lambda Runtime](Lambda-Runtime.md) | How Lambda runs code in different languages |
| [Lambda Layers](Lambda-Layers.md) | How to share libraries and dependencies |
| [Lambda Configuration](Lambda-Configuration.md) | Memory, timeout, environment variables, and roles |
| [Lambda Limits](Lambda-Limits.md) | Important quotas and things to plan for |

## Simple example

Imagine that a file is uploaded to an S3 bucket. The upload event can invoke a Lambda function. The function can then process the file automatically.

```text
User uploads file
       |
       v
    S3 bucket
       |
       | Object-created event
       v
  Lambda function
       |
       v
Process the file
```

## Why DevOps Engineers use Lambda

- Automate routine AWS tasks.
- Start or stop EC2 instances on a schedule.
- Create and manage EBS snapshots.
- React to S3, SQS, SNS, and EventBridge events.
- Build event-driven automation.
- Send alerts and trigger remediation workflows.

## Suggested learning order

1. Understand what Lambda is.
2. Learn the architecture and execution model.
3. Understand runtimes and layers.
4. Practise function configuration.
5. Learn the service quotas and design around them.

## Useful links

- [AWS Lambda documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [AWS Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/)

> **Note:** AWS quotas and supported runtimes can change. Check the official documentation before using a value in a production design.
