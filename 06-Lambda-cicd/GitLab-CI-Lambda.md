# Deploy AWS Lambda with GitLab CI/CD

GitLab CI/CD uses `.gitlab-ci.yml` to define build, test, package, and deploy stages.

## Pipeline flow

```text
Commit
  |
  v
Test
  |
  v
Package ZIP
  |
  v
Assume AWS Role
  |
  v
Deploy Lambda
```

## AWS authentication

Prefer GitLab OIDC ID tokens and AWS `AssumeRoleWithWebIdentity` rather than long-lived AWS access keys.

Configure an AWS IAM OIDC provider for your GitLab issuer. The issuer and audience must match the token configuration used by your GitLab project. Restrict the role trust policy to the intended project, namespace, branch, and audience using the claims supported by your GitLab instance.

## Example `.gitlab-ci.yml`

This example expects a GitLab runner with Python, ZIP, and AWS CLI available. It uses an OIDC token to obtain temporary AWS credentials.

```yaml
stages:
  - test
  - package
  - deploy

variables:
  AWS_REGION: "ap-south-1"
  FUNCTION_NAME: "demo-lambda"
  AWS_ROLE_ARN: "arn:aws:iam::123456789012:role/gitlab-lambda-deploy"

test:
  stage: test
  image: python:3.12
  script:
    - pip install -r requirements.txt
    - pip install pytest
    - pytest -q

package:
  stage: package
  image: python:3.12
  script:
    - rm -rf package function.zip
    - mkdir package
    - pip install -r requirements.txt -t package
    - cp -r src/* package/
    - cd package && apt-get update && apt-get install -y zip
    - zip -r ../function.zip .
  artifacts:
    paths:
      - function.zip
    expire_in: 1 day

deploy:
  stage: deploy
  image: amazon/aws-cli:2
  id_tokens:
    GITLAB_OIDC_TOKEN:
      aud: sts.amazonaws.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  script:
    - >
      CREDS=$(aws sts assume-role-with-web-identity
      --role-arn "$AWS_ROLE_ARN"
      --role-session-name "gitlab-lambda-${CI_PIPELINE_ID}"
      --web-identity-token "$GITLAB_OIDC_TOKEN"
      --duration-seconds 3600)
    - export AWS_ACCESS_KEY_ID=$(echo "$CREDS" | python -c "import sys,json; print(json.load(sys.stdin)['Credentials']['AccessKeyId'])")
    - export AWS_SECRET_ACCESS_KEY=$(echo "$CREDS" | python -c "import sys,json; print(json.load(sys.stdin)['Credentials']['SecretAccessKey'])")
    - export AWS_SESSION_TOKEN=$(echo "$CREDS" | python -c "import sys,json; print(json.load(sys.stdin)['Credentials']['SessionToken'])")
    - aws lambda update-function-code --function-name "$FUNCTION_NAME" --zip-file fileb://function.zip --region "$AWS_REGION"
    - aws lambda wait function-updated --function-name "$FUNCTION_NAME" --region "$AWS_REGION"
  dependencies:
    - package
```

For a production pipeline, use a deploy image that includes the tools you need (AWS CLI and a JSON parser such as Python), and verify that the package artifact is downloaded into the job workspace. The example's package job may be simplified by using a prebuilt image containing both Python and ZIP rather than installing OS packages during every run.

## IAM permissions

Grant the assumed role only the required Lambda deployment actions, scoped to the target function. Add `lambda:PublishVersion` and `lambda:UpdateAlias` only if your release flow uses versions and aliases.

## Best practices

- Protect the production branch and configure protected environments/approvals.
- Keep the OIDC trust policy narrow.
- Store build artifacts between stages instead of rebuilding during deployment.
- Run tests and scans before deployment.
- Keep credentials temporary and never print tokens or credential JSON in logs.

## Troubleshooting

- `AccessDenied`: check OIDC issuer, audience, subject claims, trust policy, and IAM permissions.
- `InvalidIdentityToken`: verify provider URL and token audience.
- Missing ZIP: check artifact paths and job dependencies.
- Import errors: inspect ZIP contents and dependency compatibility.
