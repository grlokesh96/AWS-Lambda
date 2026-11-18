# AWS Lambda VPC Connectivity Troubleshooting

A Lambda function attached to a VPC can access private resources such as RDS databases and internal services. Incorrect routes, security groups, DNS, or endpoints can cause connection failures.

## Typical symptoms

- Database connection timeout.
- Cannot connect to a private API.
- Cannot reach Secrets Manager or another AWS service through the expected private path.
- Function times out after VPC configuration changes.

## Network flow

```text
Lambda Network Interface
          |
          v
      Subnet Route
          |
          +---- Private RDS / Internal Service
          |
          +---- VPC Endpoint ---- AWS Service
          |
          +---- NAT Gateway ---- Public Internet
```

## Step 1: Inspect VPC configuration

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query VpcConfig
```

Confirm the intended subnet and security group IDs.

## Step 2: Check security groups

For a Lambda function connecting to PostgreSQL:

- Lambda security group outbound: allow TCP 5432 to the database security group.
- Database security group inbound: allow TCP 5432 from the Lambda security group.

Do not expose the database port to `0.0.0.0/0`.

## Step 3: Check route tables

- Confirm subnet route-table associations.
- For internet egress from private subnets, check the route to a NAT gateway.
- For private AWS service access, verify the appropriate VPC endpoint and its policy.
- Confirm DNS resolution settings and endpoint private DNS configuration where relevant.

## Step 4: Check network ACLs

Network ACLs are stateless. Verify both request and return traffic are permitted, including the required ephemeral ports.

## Step 5: Check subnet IP capacity

Lambda needs available subnet IP addresses to create and manage network interfaces. Check subnet address utilization and available IPs.

## Common misconceptions

- Attaching Lambda to a public subnet does not give it a public IP address.
- A NAT gateway provides outbound internet connectivity; it does not permit unsolicited inbound connections.
- A VPC endpoint policy does not replace the Lambda execution role's IAM permissions.

## Best practices

- Use private subnets for private application resources.
- Use security-group references rather than broad CIDR rules where possible.
- Use VPC endpoints for private AWS service access when appropriate.
- Monitor VPC Flow Logs.
- Keep database connection counts within database capacity.
