# Jenkins Pipeline for AWS Lambda

Use Jenkins to test, package, and deploy a Lambda function from a source repository.

## Pipeline stages

1. Checkout source code.
2. Install dependencies.
3. Run linting and unit tests.
4. Create a deployment ZIP.
5. Authenticate to AWS using an assumed IAM role.
6. Update the Lambda function code.
7. Optionally publish a version and update an alias.

## Example project

```text
lambda-app/
├── src/
│   └── lambda_function.py
├── tests/
│   └── test_lambda_function.py
├── requirements.txt
└── Jenkinsfile
```

Example handler:

```python
def lambda_handler(event, context):
    return {"statusCode": 200, "body": "Hello from Lambda"}
```

## Jenkinsfile example

This example assumes the Jenkins agent has Python, AWS CLI, and an approved AWS authentication mechanism configured. Prefer an instance/task role or short-lived role assumption over static access keys.

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        FUNCTION_NAME = 'demo-lambda'
        PACKAGE = 'function.zip'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install and Test') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install -r requirements.txt
                    pip install pytest
                    pytest -q
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    rm -rf package function.zip
                    mkdir package
                    pip install -r requirements.txt -t package
                    cp -r src/* package/
                    cd package
                    zip -r ../function.zip .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    aws lambda update-function-code \
                      --function-name "$FUNCTION_NAME" \
                      --zip-file "fileb://$PACKAGE" \
                      --region "$AWS_REGION"
                    aws lambda wait function-updated \
                      --function-name "$FUNCTION_NAME" \
                      --region "$AWS_REGION"
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'function.zip', allowEmptyArchive: true
        }
    }
}
```

## IAM permissions

The deployment role typically needs only the actions required by the pipeline, such as:

- `lambda:UpdateFunctionCode`
- `lambda:GetFunction`
- `lambda:GetFunctionConfiguration`
- `lambda:PublishVersion` if publishing versions
- `lambda:UpdateAlias` if moving an alias

Scope permissions to the intended function ARN. Additional permissions may be needed for artifact storage, KMS, or deployment services.

## Important notes

- Configure AWS identity on the Jenkins agent using a supported role-assumption pattern.
- Do not place AWS secret keys directly in the Jenkinsfile.
- Package native dependencies for the Lambda runtime's operating system and architecture.
- Add a deployment approval before production.
- Add smoke tests after deployment.

## Troubleshooting

| Problem | What to check |
|---|---|
| AccessDenied | IAM role trust policy and deployment permissions |
| InvalidParameterValue | ZIP layout, handler name, runtime, and architecture |
| Function update still in progress | Wait for the update to finish before another update |
| ImportModuleError | Dependency packaging and compatible build environment |
| Timeout | Function timeout, downstream dependencies, and test event |

## Interview questions

1. How do you securely authenticate Jenkins to AWS?
2. Why should dependencies be packaged for the Lambda runtime environment?
3. How would you roll back a failed Lambda deployment?
