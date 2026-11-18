# AWS Lambda Container Image Security

Lambda supports container images stored in Amazon Elastic Container Registry (ECR). Container-based functions need image security controls in addition to normal Lambda IAM and application security.

## Secure image flow

```text
Source Code
    |
    v
Docker Build
    |
    v
Dependency / Image Scan
    |
    v
Push to Private ECR
    |
    v
Deploy Lambda Image
    |
    v
Monitor and Rebuild Regularly
```

## Example Dockerfile

This example uses the AWS Lambda Python base image:

```dockerfile
FROM public.ecr.aws/lambda/python:3.12

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py ${LAMBDA_TASK_ROOT}

CMD ["app.lambda_handler"]
```

Use a runtime base image that is supported by Lambda and compatible with your function architecture.

## Build and scan

```bash
docker build -t demo-lambda:latest .

trivy image --severity HIGH,CRITICAL --exit-code 1 demo-lambda:latest
```

Review the findings and apply fixes. Avoid treating every scanner finding as exploitable without context, but do not ignore high-risk vulnerabilities without an approved exception.

## Push to ECR

Create a repository:

```bash
aws ecr create-repository \
  --repository-name demo-lambda \
  --image-scanning-configuration scanOnPush=true \
  --region ap-south-1
```

Authenticate and push:

```bash
aws ecr get-login-password --region ap-south-1 \
  | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com

docker tag demo-lambda:latest \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/demo-lambda:release-1

docker push \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/demo-lambda:release-1
```

Prefer immutable, traceable image tags or digests over relying on a mutable `latest` tag.

## Create or update Lambda

```bash
aws lambda create-function \
  --function-name demo-container-lambda \
  --package-type Image \
  --code ImageUri=123456789012.dkr.ecr.ap-south-1.amazonaws.com/demo-lambda:release-1 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --region ap-south-1
```

For an existing image-based function, update the image URI with `update-function-code --image-uri ...`.

## Security best practices

- Use minimal, trusted base images and rebuild regularly.
- Scan images before pushing and enable ECR image scanning.
- Do not bake secrets into image layers or build arguments.
- Run the function with a least-privilege execution role.
- Restrict ECR repository access to required principals.
- Use image digests or immutable tags for release traceability.
- Keep the Lambda image architecture compatible with the function configuration.
- Rebuild and redeploy when base images or dependencies receive security fixes.

## Important note

Lambda container images are not the same as running a general-purpose container on ECS or EKS. Follow Lambda's image requirements and supported image size, architecture, and runtime constraints.
