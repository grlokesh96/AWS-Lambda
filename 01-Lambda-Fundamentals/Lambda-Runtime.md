# AWS Lambda Runtime

## 1. Introduction

A runtime is the language environment that AWS Lambda uses to execute your function code.

For example, if you write a function in Python, Lambda uses a Python runtime to execute it.

## 2. Common runtime languages

AWS Lambda supports several managed runtimes. Examples include:

- Python
- Node.js
- Java
- .NET
- Ruby

The exact supported versions change over time. Always check the current AWS documentation before selecting a runtime.

Official reference: [Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)

## 3. How the runtime works

```text
Your function code
       |
       v
Lambda runtime
 (Python / Node.js /
  Java / etc.)
       |
       v
Lambda execution environment
       |
       v
Function executes
```

The runtime receives invocation information and runs the configured handler.

## 4. Python Lambda example

Create a file named `lambda_function.py`.

```python
def lambda_handler(event, context):
    name = event.get("name", "DevOps Engineer")

    return {
        "statusCode": 200,
        "body": f"Hello, {name}!"
    }
```

Example test event:

```json
{
  "name": "Lokesh"
}
```

Example response:

```json
{
  "statusCode": 200,
  "body": "Hello, Lokesh!"
}
```

In this example:

- `lambda_function.py` is the file name.
- `lambda_handler` is the handler function.
- `event` contains the input.
- `context` contains invocation information.

The configured handler is commonly written as:

```text
lambda_function.lambda_handler
```

## 5. Managed runtime vs custom runtime

| Type | Explanation |
|---|---|
| Managed runtime | AWS provides and maintains the language runtime |
| Custom runtime | You provide the runtime interface needed to execute your code |

Managed runtimes are usually simpler for standard DevOps automation tasks.

Custom runtimes are useful when you need a runtime environment that is not provided as a managed option.

## 6. Runtime selection tips

- Choose a runtime supported by AWS.
- Use a version compatible with your application dependencies.
- Test runtime upgrades before production deployment.
- Monitor runtime deprecation announcements.
- Package dependencies correctly.

## 7. DevOps use case

A DevOps Engineer can use Python with Boto3 to automate AWS tasks.

Examples:

- Start and stop EC2 instances.
- Create EBS snapshots.
- Clean up old ECR images.
- Check unused resources.
- Send CloudWatch alerts.

## 8. Interview questions

**Q1. What is a Lambda runtime?**

A runtime provides the language environment required to execute Lambda function code.

**Q2. What is a Lambda handler?**

It is the entry point that Lambda invokes to execute the function.

**Q3. Can we use Python in Lambda?**

Yes. AWS provides managed Python runtimes, subject to the currently supported versions.

## Summary

The runtime executes your code, while the handler is the function entry point that processes each invocation.
