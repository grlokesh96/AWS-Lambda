# Project 07: Serverless CI/CD Pipeline

## Objective

Build, test, scan, package, and deploy an AWS Lambda function automatically using GitHub Actions and AWS SAM.

## Architecture

```text
Developer push / Pull Request
             |
             v
       GitHub Actions
             |
      +------+------+
      |             |
   Test + lint   Security scan
      |             |
      +------+------+
             |
         SAM build
             |
      Configure AWS OIDC
             |
             v
       SAM deploy
             |
             v
       AWS Lambda
```

## Suggested repository structure

```text
serverless-app/
├── src/
│   └── app.py
├── tests/
│   └── test_app.py
├── template.yaml
├── requirements.txt
└── .github/
    └── workflows/
        └── deploy.yml
```

## Minimal SAM template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Example serverless application

Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Runtime: python3.12
      Timeout: 10
      MemorySize: 256
      Policies:
        - AWSLambdaBasicExecutionRole
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

Check the currently supported Lambda runtimes before choosing one for a new deployment.

## Example Lambda handler

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": "Hello from Lambda"})
    }
```

## GitHub Actions workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Serverless CI/CD

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: |
          python -m pip install --upgrade pip
          pip install pytest
      - run: pytest -q

  deploy:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - uses: aws-actions/setup-sam@v2
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}
      - name: Build
        run: sam build
      - name: Deploy
        run: |
          sam deploy \
            --stack-name serverless-demo \
            --resolve-s3 \
            --capabilities CAPABILITY_IAM \
            --no-confirm-changeset \
            --no-fail-on-empty-changeset
```

For production, pin third-party actions to reviewed commit SHAs, add linting and dependency/security scans, and use separate AWS roles and environments for dev/stage/prod.

## OIDC configuration

1. Add GitHub as an OIDC identity provider in IAM (`token.actions.githubusercontent.com`).
2. Create a role with a trust policy restricted to your organization, repository, and branch or GitHub Environment.
3. Grant only deployment permissions required by the CloudFormation/SAM stack.
4. Store the role ARN in the repository variable or secret `AWS_DEPLOY_ROLE_ARN`.
5. Configure `AWS_REGION` as a repository or environment variable.

Do not store long-lived AWS access keys in GitHub secrets when OIDC federation is available.

## Useful commands

```bash
sam validate
sam build
sam local invoke HelloFunction --event events/event.json
sam deploy --guided
aws cloudformation describe-stacks --stack-name serverless-demo
```

## Deployment best practices

- Use pull requests for review and test before deployment.
- Separate build/test from deployment.
- Use CloudFormation change sets and approval gates for production.
- Deploy through aliases and controlled traffic shifting for critical functions.
- Keep environment-specific configuration outside source code.
- Add rollback, alarms, and post-deployment smoke tests.
