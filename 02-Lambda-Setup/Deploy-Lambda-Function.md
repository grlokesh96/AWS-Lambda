# Deploy an AWS Lambda Function

## 1. Introduction

Deployment means uploading your function code and configuration to AWS Lambda so it can run.

You can deploy Lambda functions using:

- AWS Management Console.
- AWS CLI.
- AWS SAM.
- AWS CloudFormation.
- Terraform.
- CI/CD tools such as GitHub Actions and Jenkins.

This guide uses AWS CLI to deploy a ZIP package.

## 2. Prerequisites

- A Lambda function already created.
- AWS CLI installed and configured.
- Permission to update the function code.
- The function's source code.

## 3. Project structure

```text
lambda-demo/
├── lambda_function.py
└── event.json
```

## 4. Update the function code

Example `lambda_function.py`:

```python
def lambda_handler(event, context):
    name = event.get("name", "DevOps Engineer")

    return {
        "statusCode": 200,
        "body": f"Hello, {name}! Function deployed successfully."
    }
```

## 5. Create a deployment package

```bash
zip -r function.zip lambda_function.py
```

For functions with third-party dependencies, package the dependencies in the correct structure or use a Lambda Layer or container image.

## 6. Deploy the package

```bash
aws lambda update-function-code \
  --function-name lambda-demo \
  --zip-file fileb://function.zip
```

The command returns information about the updated function.

## 7. Wait for the update to finish

```bash
aws lambda wait function-updated-v2 \
  --function-name lambda-demo
```

Then check the function configuration:

```bash
aws lambda get-function-configuration \
  --function-name lambda-demo \
  --query '{State:State,LastUpdateStatus:LastUpdateStatus,Runtime:Runtime}'
```

## 8. Update function configuration

For example, change the timeout:

```bash
aws lambda update-function-configuration \
  --function-name lambda-demo \
  --timeout 30
```

Change the memory:

```bash
aws lambda update-function-configuration \
  --function-name lambda-demo \
  --memory-size 256
```

Wait for configuration updates to finish before making another configuration update.

## 9. Deployment using AWS Console

1. Open AWS Lambda.
2. Select your function.
3. Open the **Code** tab.
4. Upload the deployment ZIP file or edit the code.
5. Choose **Deploy** when using the code editor.
6. Test the function.

## 10. Basic deployment flow

```text
Developer changes code
        |
        v
Run tests
        |
        v
Create ZIP package
        |
        v
Upload package using AWS CLI
        |
        v
Wait for update
        |
        v
Invoke function
        |
        v
Verify logs and output
```

## 11. Deployment best practices

- Test code before deployment.
- Keep development, test, and production environments separate.
- Use CI/CD instead of manual production deployments.
- Use IAM roles or OIDC federation for CI/CD authentication.
- Store deployment artifacts securely.
- Use Lambda versions and aliases for controlled releases.
- Use gradual deployment strategies when appropriate.
- Keep a rollback plan.

## 12. Common errors

| Error | What to check |
|---|---|
| ResourceNotFoundException | Function name and AWS region |
| AccessDeniedException | Deployment identity permissions |
| InvalidParameterValueException | Runtime, handler, or package configuration |
| Function update still in progress | Wait for the previous update to finish |
| ImportModuleError | Dependency and ZIP package structure |

## 13. Interview questions

**Q1. How can you deploy Lambda code?**

Using the AWS Console, AWS CLI, SAM, CloudFormation, Terraform, or a CI/CD pipeline.

**Q2. How do you update Lambda code using AWS CLI?**

Use `aws lambda update-function-code`.

**Q3. How do you automate Lambda deployments?**

Use a CI/CD pipeline such as GitHub Actions or Jenkins with secure AWS authentication.

## Summary

A Lambda deployment package contains your code and required dependencies. Upload it to the function, wait for the update, and test the deployed version.
