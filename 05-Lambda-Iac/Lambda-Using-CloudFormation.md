# Deploy AWS Lambda Using CloudFormation

## 1. Introduction

AWS CloudFormation is an Infrastructure as Code service provided by AWS.

It allows you to define AWS resources in YAML or JSON templates and deploy them as a stack.

We can use CloudFormation to create Lambda functions, IAM roles, permissions, and triggers.

## 2. Architecture

```text
CloudFormation Template
          |
          v
     CloudFormation
          |
       +--+--+
       |     |
       v     v
      IAM  Lambda
       |     |
       +--+--+
          |
          v
    CloudWatch Logs
```

## 3. Prerequisites

- AWS CLI installed.
- AWS credentials configured.
- Python function code.
- Permission to create CloudFormation stacks and required resources.

## 4. Project structure

```text
lambda-cloudformation/
├── template.yaml
└── lambda_function.py
```

## 5. Create Lambda code

Create `lambda_function.py`:

```python
def lambda_handler(event, context):
    name = event.get("name", "DevOps Engineer")

    return {
        "statusCode": 200,
        "body": f"Hello {name} from CloudFormation!"
    }
```

## 6. Package the function

Create a ZIP file:

```bash
zip function.zip lambda_function.py
```

Upload it to an S3 bucket used for deployment artifacts:

```bash
aws s3 cp function.zip s3://YOUR-DEPLOYMENT-BUCKET/lambda/function.zip
```

The bucket must be in the same AWS Region as the stack for this S3 artifact reference.

## 7. Create the CloudFormation template

Create `template.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Create a basic Python Lambda function

Parameters:
  LambdaCodeBucket:
    Type: String

  LambdaCodeKey:
    Type: String

Resources:
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

  DemoLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: cloudformation-lambda-demo
      Runtime: python3.12
      Handler: lambda_function.lambda_handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        S3Bucket: !Ref LambdaCodeBucket
        S3Key: !Ref LambdaCodeKey
      Timeout: 30
      MemorySize: 256

Outputs:
  LambdaFunctionArn:
    Description: Lambda function ARN
    Value: !GetAtt DemoLambdaFunction.Arn
```

Select a Python runtime currently supported by Lambda. The example's runtime may need updating over time.

## 8. Validate the template

```bash
aws cloudformation validate-template \
  --template-body file://template.yaml
```

This checks template syntax and structure. It does not guarantee that every resource deployment will succeed.

## 9. Deploy the stack

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name lambda-demo-stack \
  --parameter-overrides \
    LambdaCodeBucket=YOUR-DEPLOYMENT-BUCKET \
    LambdaCodeKey=lambda/function.zip \
  --capabilities CAPABILITY_IAM
```

CloudFormation uses the IAM role resource in this template, so the IAM capability must be acknowledged.

## 10. Verify the stack

```bash
aws cloudformation describe-stacks \
  --stack-name lambda-demo-stack
```

List stack resources:

```bash
aws cloudformation list-stack-resources \
  --stack-name lambda-demo-stack
```

## 11. Test Lambda

```bash
aws lambda invoke \
  --function-name cloudformation-lambda-demo \
  --payload '{"name":"Lokesh"}' \
  response.json
```

View the response:

```bash
cat response.json
```

## 12. Delete the stack

When the practice is complete:

```bash
aws cloudformation delete-stack \
  --stack-name lambda-demo-stack
```

Wait for deletion:

```bash
aws cloudformation wait stack-delete-complete \
  --stack-name lambda-demo-stack
```

The deployment artifact in S3 is not automatically deleted by this stack because it is not a resource managed by the template.

## 13. Best practices

- Store templates in Git.
- Use parameters for environment-specific values.
- Review change sets before applying significant changes.
- Use separate stacks for environments where appropriate.
- Protect sensitive data.
- Apply least-privilege IAM permissions.
- Use CloudFormation StackSets when managing supported deployments across multiple accounts and regions.

## 14. Interview questions

**Q1. What is CloudFormation?**

It is an AWS service for creating and managing infrastructure using templates.

**Q2. What is a CloudFormation stack?**

A stack is a collection of AWS resources managed together using a template.

**Q3. How do you deploy Lambda using CloudFormation?**

Package the function code, upload it to S3, reference the artifact in the template, and deploy the stack.

## Summary

CloudFormation lets you create and manage Lambda infrastructure using native AWS templates.
