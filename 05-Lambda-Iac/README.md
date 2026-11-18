# AWS Lambda Infrastructure as Code (IaC)

Infrastructure as Code (IaC) means creating and managing cloud resources using configuration files or code instead of creating everything manually in the AWS Console.

For Lambda, IaC can manage the function, IAM execution role, permissions, triggers, environment variables, and related resources.

## Folder contents

| File | What you will learn |
|---|---|
| [Lambda Using Terraform](Lambda-Using-Terraform.md) | Create and deploy Lambda using Terraform |
| [Lambda Using CloudFormation](Lambda-Using-CloudFormation.md) | Create Lambda resources using a CloudFormation template |
| [Lambda Using CDK](Lambda-Using-CDK.md) | Define Lambda infrastructure using AWS CDK and Python |
| [Lambda Environment Variables](Lambda-Environment-Variables.md) | Configure and manage function environment variables |

## What is IaC?

Instead of creating resources manually, you define the required infrastructure in code.

Example:

```text
Terraform / CloudFormation / CDK
               |
               v
        AWS infrastructure
               |
       +-------+-------+
       |       |       |
       v       v       v
    Lambda    IAM    EventBridge
```

## Why use IaC for Lambda?

- Create repeatable environments.
- Keep infrastructure changes in Git.
- Review changes before deployment.
- Reuse configuration across environments.
- Reduce manual configuration errors.
- Support automated CI/CD deployments.
- Track infrastructure changes.

## IaC tools

| Tool | Description |
|---|---|
| Terraform | Declarative infrastructure provisioning tool |
| CloudFormation | Native AWS infrastructure provisioning service |
| AWS CDK | Define cloud infrastructure using programming languages |

## Recommended workflow

1. Write the infrastructure code.
2. Review the configuration.
3. Validate or synthesize the template.
4. Review the planned changes.
5. Deploy to a development environment.
6. Test the Lambda function.
7. Promote the approved changes to higher environments.

## Important security practices

- Use IAM roles with least privilege.
- Never hardcode AWS access keys.
- Store Terraform state securely.
- Protect state files because they may contain sensitive values.
- Avoid putting secrets in plain-text configuration.
- Review infrastructure changes before applying them.
- Use separate environments and deployment permissions.

## Official documentation

- [Terraform AWS Lambda resources](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lambda_function)
- [AWS CloudFormation Lambda](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_Lambda.html)
- [AWS CDK documentation](https://docs.aws.amazon.com/cdk/v2/guide/home.html)
- [Lambda environment variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
