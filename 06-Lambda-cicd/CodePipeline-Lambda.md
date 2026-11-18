# AWS CodePipeline for Lambda

AWS CodePipeline can orchestrate an AWS-native Lambda delivery workflow. A common setup uses CodeCommit or a supported external source connection, CodeBuild for tests and packaging, and a deployment action or CodeDeploy for controlled releases.

## Architecture

```text
Source Repository
      |
      v
 AWS CodePipeline
      |
      v
   CodeBuild
   - Install dependencies
   - Run tests
   - Package ZIP
      |
      v
 Deploy Lambda
      |
      v
 Publish Version / Shift Alias Traffic
      |
      v
 CloudWatch Monitoring
```

## Example CodeBuild `buildspec.yml`

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - python -m pip install --upgrade pip
      - pip install -r requirements.txt
      - pip install pytest

  pre_build:
    commands:
      - pytest -q

  build:
    commands:
      - rm -rf package function.zip
      - mkdir package
      - pip install -r requirements.txt -t package
      - cp -r src/* package/
      - cd package
      - zip -r ../function.zip .
      - cd ..

artifacts:
  files:
    - function.zip
```

## Deployment options

### Direct CodePipeline deployment

A pipeline action can invoke a deployment step that updates the Lambda function code. Ensure the action's role has the necessary Lambda permissions and that the artifact ZIP is available to the action.

### CodeDeploy traffic shifting

For Lambda deployments that need canary, linear, or all-at-once traffic shifting, integrate CodeDeploy with Lambda aliases. CodeDeploy shifts traffic between Lambda versions and can automatically roll back based on configured alarms.

## IAM roles

Separate roles by responsibility:

- CodePipeline service role: access source, invoke CodeBuild, and pass required roles.
- CodeBuild service role: read build inputs, write artifacts, and access only required build resources.
- Deployment role: update Lambda code, publish versions, and update aliases as required.

Use least privilege and restrict `iam:PassRole` to the exact service role ARN when needed.

## Deployment checklist

- [ ] Source connection is configured.
- [ ] Buildspec runs tests and creates the expected ZIP.
- [ ] Artifact is stored and passed to the deploy stage.
- [ ] IAM roles are least-privilege.
- [ ] Production approval is configured where appropriate.
- [ ] Post-deployment alarms and rollback behavior are tested.

## Key point

CodePipeline orchestrates stages; CodeBuild performs build commands; Lambda versions and aliases provide release control. CodeDeploy is the AWS service commonly used for managed Lambda traffic shifting.
