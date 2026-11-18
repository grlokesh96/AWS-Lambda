# Deploy AWS Lambda Using AWS CDK

## 1. Introduction

AWS CDK (Cloud Development Kit) lets you define cloud infrastructure using programming languages.

You can use Python, TypeScript, Java, and other supported languages to define AWS resources.

CDK converts your code into a CloudFormation template and uses CloudFormation to deploy the infrastructure.

## 2. Architecture

```text
AWS CDK Application
        |
        v
   CDK Synthesis
        |
        v
CloudFormation Template
        |
        v
 CloudFormation Stack
        |
        v
   Lambda Function
```

## 3. Prerequisites

Install:

- Python 3.
- Node.js and npm.
- AWS CLI.
- AWS CDK CLI.
- An AWS identity with deployment permissions.

Install CDK CLI:

```bash
npm install -g aws-cdk
```

Verify:

```bash
cdk --version
aws sts get-caller-identity
```

## 4. Create a CDK project

```bash
mkdir lambda-cdk
cd lambda-cdk
```

Initialize a Python CDK application:

```bash
cdk init app --language python
```

Activate the virtual environment created by the project:

```bash
source .venv/bin/activate
```

Install project dependencies:

```bash
pip install -r requirements.txt
```

## 5. Create Lambda code

Create a directory:

```bash
mkdir lambda_src
```

Create `lambda_src/lambda_function.py`:

```python
def lambda_handler(event, context):
    name = event.get("name", "DevOps Engineer")

    return {
        "statusCode": 200,
        "body": f"Hello {name} from AWS CDK!"
    }
```

## 6. Define Lambda infrastructure

Open the generated stack file, usually `lambda_cdk/lambda_cdk_stack.py`, and replace its contents with:

```python
from aws_cdk import (
    Stack,
    Duration,
    aws_lambda as _lambda,
)
from constructs import Construct

class LambdaCdkStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs):
        super().__init__(scope, construct_id, **kwargs)

        _lambda.Function(
            self,
            "DemoLambdaFunction",
            function_name="cdk-lambda-demo",
            runtime=_lambda.Runtime.PYTHON_3_12,
            handler="lambda_function.lambda_handler",
            code=_lambda.Code.from_asset("lambda_src"),
            memory_size=256,
            timeout=Duration.seconds(30)
        )
```

Use a runtime supported by Lambda and by the CDK version installed in your project. Update the runtime constant when necessary.

## 7. Synthesize the template

```bash
cdk synth
```

This generates the CloudFormation template from your CDK application.

## 8. Bootstrap the AWS environment

For an AWS account and region that have not been bootstrapped for CDK:

```bash
cdk bootstrap aws://ACCOUNT_ID/REGION
```

Replace the placeholders with your AWS account ID and target region.

Bootstrapping creates resources that CDK uses for deployments. Review the bootstrap permissions and trust configuration in production environments.

## 9. Deploy the Lambda function

```bash
cdk deploy
```

Review the proposed changes and approve deployment.

## 10. Verify deployment

```bash
aws lambda get-function \
  --function-name cdk-lambda-demo
```

## 11. Test Lambda

```bash
aws lambda invoke \
  --function-name cdk-lambda-demo \
  --payload '{"name":"Lokesh"}' \
  response.json
```

Check the response:

```bash
cat response.json
```

## 12. Destroy the CDK stack

When the practice is complete:

```bash
cdk destroy
```

Review the resources before confirming. CDK bootstrap resources are separate and are not normally removed by destroying an individual application stack.

## 13. Best practices

- Keep infrastructure code in Git.
- Use reusable constructs.
- Separate application code from infrastructure code.
- Review synthesized CloudFormation templates.
- Use environment-specific configuration.
- Use secure deployment identities.
- Apply least-privilege permissions.
- Test changes in development before production.

## 14. Interview questions

**Q1. What is AWS CDK?**

It is a framework for defining cloud infrastructure using programming languages.

**Q2. Does CDK use CloudFormation?**

Yes. CDK synthesizes CloudFormation templates and deploys through CloudFormation.

**Q3. What is `cdk synth`?**

It generates the CloudFormation template from the CDK application.

**Q4. What is `cdk bootstrap`?**

It prepares an AWS account and region for CDK deployments.

## Summary

AWS CDK allows developers and DevOps Engineers to define Lambda infrastructure using familiar programming languages.
