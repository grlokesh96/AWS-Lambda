# AWS Lambda VPC Security

A Lambda function does not need a VPC just to access public AWS services. Attach it to a VPC when it must reach private resources such as an internal database, private service, or private endpoint.

## Architecture

```text
                  Lambda Function
                         |
                    ENI / VPC
                         |
                Private Subnets
                  /          \
             Security       Route Tables
              Groups            |
                  |         NAT / Endpoints
                  |
             Private RDS
```

## Main security components

- **Private subnets:** Place Lambda network interfaces in subnets that do not assign public IP addresses.
- **Security groups:** Control inbound and outbound traffic for the function's network interfaces.
- **Network ACLs:** Provide subnet-level stateless filtering.
- **VPC endpoints:** Allow private connectivity to supported AWS services without routing through the public internet.
- **NAT gateway:** Provides outbound internet access from private subnets when required. It does not provide unsolicited inbound access.

## Example security group design

For a Lambda function connecting to an RDS database:

- Lambda security group outbound: allow TCP 5432 to the database security group.
- Database security group inbound: allow TCP 5432 from the Lambda security group.
- Avoid opening the database port to `0.0.0.0/0`.

Use the actual database port for your engine.

## Configure VPC access

A Lambda VPC configuration requires subnet IDs and security group IDs. Example:

```bash
aws lambda update-function-configuration \
  --function-name my-vpc-lambda \
  --vpc-config SubnetIds=subnet-11111111,subnet-22222222,SecurityGroupIds=sg-12345678 \
  --region ap-south-1
```

The Lambda execution role needs the permissions required for Lambda to manage its VPC network interfaces. The AWS-managed `AWSLambdaVPCAccessExecutionRole` policy is a common starting point; review its current permissions.

## Private access to AWS services

Consider interface or gateway VPC endpoints where supported, for example:

- S3 gateway endpoint.
- DynamoDB gateway endpoint.
- Secrets Manager interface endpoint.
- CloudWatch Logs interface endpoint, if private log delivery is required by your design.

Use endpoint policies to restrict which resources can be accessed through an endpoint.

## Security best practices

- Use private subnets and narrowly scoped security groups.
- Do not expose databases directly to the internet.
- Use TLS for service-to-service communication where supported.
- Use VPC endpoints for private access to AWS services when appropriate.
- Keep routing and endpoint policies minimal.
- Monitor VPC Flow Logs and application connection errors.
- Remember that attaching Lambda to a public subnet does not give the function a public IP or direct internet access.

## Troubleshooting

| Symptom | Check |
|---|---|
| Database connection timeout | Routes, security groups, NACLs, database availability, and port |
| No internet egress | NAT gateway, route table, and subnet association |
| Cannot reach Secrets Manager | DNS, endpoint routes/security group, endpoint policy, IAM role |
| Function times out after VPC change | ENI setup, subnet IP availability, routing, and security rules |
