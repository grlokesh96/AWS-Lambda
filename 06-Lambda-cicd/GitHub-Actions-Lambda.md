# Deploy AWS Lambda with GitHub Actions

GitHub Actions can automate testing, packaging, and deployment of Lambda functions whenever code is pushed or a release is created.

## Recommended authentication: GitHub OIDC

Use GitHub's OpenID Connect integration to assume an AWS IAM role. This avoids storing long-lived AWS access keys as GitHub secrets.

### 1. Configure an AWS IAM OIDC provider

In the AWS account, create or verify the GitHub OIDC provider:

- Provider URL: `https://token.actions.githubusercontent.com`
- Audience: `sts.amazonaws.com`

### 2. Create an IAM role

The role trust policy should restrict access to the intended repository and branch. Example trust-policy condition:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
        "token.actions.githubusercontent.com:sub": "repo:OWNER/REPOSITORY:ref:refs/heads/main"
      }
    }
  }]
}
```

Replace the account ID, owner, and repository. For GitHub environments, the `sub` claim format differs; configure the trust condition to match the exact environment subject.

Attach a least-privilege deployment policy to the role. For a basic ZIP deployment, the role may need `lambda:UpdateFunctionCode` and read permissions for the target function. Add version and alias permissions only if the workflow uses them.

## 3. Add the workflow

Create `.github/workflows/deploy-lambda.yml`:

```yaml
name: Lambda CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install test dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest

      - name: Run tests
        run: pytest -q

  deploy:
    if: github.event_name != 'pull_request'
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    env:
      AWS_REGION: ap-south-1
      FUNCTION_NAME: demo-lambda
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Build deployment package
        run: |
          rm -rf package function.zip
          mkdir package
          pip install -r requirements.txt -t package
          cp -r src/* package/
          cd package
          zip -r ../function.zip .

      - name: Configure AWS credentials with OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-lambda-deploy
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy Lambda
        run: |
          aws lambda update-function-code \
            --function-name "$FUNCTION_NAME" \
            --zip-file fileb://function.zip
          aws lambda wait function-updated \
            --function-name "$FUNCTION_NAME"
```

Pin third-party actions to reviewed full commit SHAs in security-sensitive repositories. The version tags above are readable examples, not a substitute for your organization's action-pinning policy.

## 4. Verify deployment

```bash
aws lambda get-function \
  --function-name demo-lambda \
  --region ap-south-1

aws lambda invoke \
  --function-name demo-lambda \
  --payload '{}' \
  response.json

cat response.json
```

## Best practices

- Use OIDC and restrict the IAM trust policy to the exact repository, branch, or GitHub environment.
- Keep deployment permissions in the deploy job only.
- Do not deploy pull-request code to production.
- Use GitHub Environments for approvals and protected deployment branches.
- Publish a Lambda version and update a production alias for controlled releases.
- Run security scans and smoke tests before and after deployment.
