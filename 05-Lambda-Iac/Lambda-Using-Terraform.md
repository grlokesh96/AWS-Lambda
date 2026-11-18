# Deploy AWS Lambda Using Terraform

## 1. Introduction

Terraform is an Infrastructure as Code tool used to create and manage cloud resources.

We can use Terraform to create:

- Lambda functions.
- IAM execution roles.
- IAM policies.
- Lambda permissions.
- EventBridge triggers.
- CloudWatch log groups.

In this example, we create a simple Python Lambda function and its IAM execution role.

## 2. Architecture

```text
Terraform configuration
          |
          v
      AWS Provider
          |
          v
      IAM Role
          |
          v
    Lambda Function
          |
          v
    CloudWatch Logs
```

## 3. Prerequisites

Install and configure:

- Terraform.
- AWS CLI.
- Python 3.
- An AWS identity with the required permissions.

Verify:

```bash
terraform version
aws --version
aws sts get-caller-identity
```

## 4. Project structure

```text
lambda-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── lambda_function.py
└── terraform.tfvars
```

## 5. Create the Lambda code

Create `lambda_function.py`:

```python
def lambda_handler(event, context):
    name = event.get("name", "DevOps Engineer")

    return {
        "statusCode": 200,
        "body": f"Hello {name} from Terraform Lambda!"
    }
```

## 6. Create Terraform configuration

Create `main.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

data "archive_file" "lambda_zip" {
  type        = "zip"
  source_file = "${path.module}/lambda_function.py"
  output_path = "${path.module}/function.zip"
}

resource "aws_iam_role" "lambda_role" {
  name = "${var.function_name}-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_logs" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_lambda_function" "demo" {
  function_name    = var.function_name
  role             = aws_iam_role.lambda_role.arn
  runtime          = var.runtime
  handler          = "lambda_function.lambda_handler"
  filename         = data.archive_file.lambda_zip.output_path
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256

  timeout     = 30
  memory_size = 256

  depends_on = [
    aws_iam_role_policy_attachment.lambda_logs
  ]
}
```

The example uses AWS provider version 6.x. Pin and test a provider version that matches your project standards.

## 7. Create variables

Create `variables.tf`:

```hcl
variable "aws_region" {
  description = "AWS region for Lambda deployment"
  type        = string
  default     = "ap-south-1"
}

variable "function_name" {
  description = "Lambda function name"
  type        = string
  default     = "terraform-lambda-demo"
}

variable "runtime" {
  description = "Supported AWS Lambda Python runtime"
  type        = string
  default     = "python3.12"
}
```

Choose a runtime that is currently supported by Lambda in your region.

## 8. Create outputs

Create `outputs.tf`:

```hcl
output "lambda_function_name" {
  value = aws_lambda_function.demo.function_name
}

output "lambda_function_arn" {
  value = aws_lambda_function.demo.arn
}

output "lambda_execution_role_arn" {
  value = aws_iam_role.lambda_role.arn
}
```

## 9. Create terraform.tfvars

```hcl
aws_region    = "ap-south-1"
function_name = "terraform-lambda-demo"
runtime       = "python3.12"
```

## 10. Initialize Terraform

```bash
terraform init
```

## 11. Format and validate

```bash
terraform fmt -recursive
terraform validate
```

## 12. Review the plan

```bash
terraform plan
```

Review the planned resources and permissions before applying.

## 13. Deploy Lambda

```bash
terraform apply
```

Enter `yes` when prompted after reviewing the plan.

## 14. Verify deployment

```bash
aws lambda get-function \
  --function-name terraform-lambda-demo
```

## 15. Invoke Lambda

```bash
aws lambda invoke \
  --function-name terraform-lambda-demo \
  --payload '{"name":"Lokesh"}' \
  response.json
```

View the result:

```bash
cat response.json
```

## 16. Clean up resources

When finished, review the destroy plan:

```bash
terraform plan -destroy
```

Then remove the practice resources:

```bash
terraform destroy
```

Do not run destroy against shared or production infrastructure without an approved change.

## 17. Best practices

- Use remote state with locking supported by your chosen backend.
- Enable encryption and access controls for state storage.
- Never commit state files or secret values to Git.
- Use modules for reusable Lambda infrastructure.
- Pin provider versions.
- Review Terraform plans before applying.
- Use separate workspaces or state configurations for environments.

## 18. Interview questions

**Q1. How do you create Lambda using Terraform?**

Define the Lambda resource, IAM role, permissions, runtime, handler, and deployment package in Terraform configuration.

**Q2. What is `source_code_hash`?**

It helps Terraform detect changes to the packaged Lambda code and trigger an update.

**Q3. Why use Terraform for Lambda?**

It makes Lambda infrastructure repeatable, version-controlled, and suitable for CI/CD automation.

## Summary

Terraform can manage the complete Lambda infrastructure lifecycle, from IAM role creation to function deployment.
