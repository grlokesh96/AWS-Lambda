# Create an AWS Lambda Function

## 1. Introduction

In this guide, we create a simple Python Lambda function.

The function receives an event and returns a greeting.

## 2. Prerequisites

- AWS CLI installed.
- AWS credentials configured.
- A supported Python Lambda runtime.
- A Lambda execution role already created.
- Permission to create Lambda functions.

Check your AWS identity:

```bash
aws sts get-caller-identity
```

Check the region:

```bash
aws configure get region
```

## 3. Create a project folder

```bash
mkdir lambda-demo
cd lambda-demo
```

## 4. Create the Python function

Create a file named `lambda_function.py`:

```python
def lambda_handler(event, context):
    name = event.get("name", "DevOps Engineer")

    return {
        "statusCode": 200,
        "body": f"Hello, {name}! Welcome to AWS Lambda."
    }
```

### Understand the code

- `lambda_handler` is the handler function.
- `event` contains the input sent to Lambda.
- `context` contains information about the current invocation.
- The function returns a Python dictionary.

## 5. Create a test event

Create `event.json`:

```json
{
  "name": "Lokesh"
}
```

## 6. Package the function

For this simple example, no third-party dependencies are required.

```bash
zip function.zip lambda_function.py
```

Check the package:

```bash
unzip -l function.zip
```

The ZIP file should contain `lambda_function.py` at its root.

## 7. Create the Lambda function using AWS CLI

First, get the execution role ARN:

```bash
aws iam get-role \
  --role-name lambda-basic-execution-role \
  --query 'Role.Arn' \
  --output text
```

Use the returned ARN in the command below.

Choose a Python runtime currently supported by Lambda. The example uses `python3.12`; change it if that runtime is no longer supported in your region.

```bash
aws lambda create-function \
  --function-name lambda-demo \
  --runtime python3.12 \
  --role <LAMBDA_EXECUTION_ROLE_ARN> \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

Replace `<LAMBDA_EXECUTION_ROLE_ARN>` with the actual role ARN. The role must be in the same AWS account.

## 8. Verify the function

```bash
aws lambda get-function \
  --function-name lambda-demo
```

Check its state:

```bash
aws lambda get-function-configuration \
  --function-name lambda-demo \
  --query '{State:State,Runtime:Runtime,Handler:Handler}'
```

A newly created function may take a short time to become active.

## 9. Create a function using the AWS Console

1. Open the AWS Management Console.
2. Search for **Lambda**.
3. Select **Create function**.
4. Choose **Author from scratch**.
5. Enter `lambda-demo` as the function name.
6. Select a supported Python runtime.
7. Select the execution role created earlier.
8. Choose **Create function**.
9. Add or edit the function code.
10. Deploy the code changes.

## 10. Common errors

| Error | What to check |
|---|---|
| Role does not exist | Check the execution role ARN and account |
| Invalid runtime | Choose a currently supported runtime |
| Handler not found | Verify the file and handler names |
| AccessDenied | Check the deployment identity's permissions |
| Function already exists | Choose another name or update the existing function |

## 11. Interview questions

**Q1. What is a Lambda function?**

It is the code and configuration that AWS Lambda executes in response to an invocation.

**Q2. What is a handler?**

It is the entry point Lambda calls when the function is invoked.

**Q3. What is the purpose of the event parameter?**

It contains the input data for the invocation.

## Summary

You have created a basic Python Lambda function. Next, package and deploy code updates, then invoke the function to verify its output.
