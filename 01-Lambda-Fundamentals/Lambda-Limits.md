# AWS Lambda Limits

## 1. Introduction

AWS Lambda has service quotas and limits. These limits help control resource usage and protect the service.

A DevOps Engineer should understand these limits before deploying production workloads.

**Important:** AWS can change quotas. Always verify the current values in the official documentation and in your AWS account.

Official reference: [AWS Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)

## 2. Important limits to understand

| Area | What to check |
|---|---|
| Function timeout | Maximum allowed execution duration |
| Memory | Minimum and maximum memory configuration |
| Deployment package | ZIP package and container image size limits |
| Ephemeral storage | Available temporary `/tmp` storage |
| Concurrency | Regional and function-level concurrency quotas |
| Layers | Maximum number of layers attached to a function |
| Environment variables | Total size allowed for environment variables |
| Invocation payload | Request and response payload limits |
| File descriptors | Operating-system resource limits |

Refer to the official quotas page for the latest numeric values and exceptions.

## 3. Timeout limit

Every function has a timeout setting.

If a function takes longer than its configured timeout, the invocation fails.

Example:

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --timeout 60
```

Increasing timeout does not fix a slow or blocked dependency. Check logs, network calls, and application performance first.

## 4. Memory limit

Lambda memory affects both memory available and CPU power.

Example:

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --memory-size 1024
```

Test different memory settings to find a good balance between execution time and cost.

## 5. Concurrency limit

Concurrency is the number of function invocations running at the same time.

For example:

```text
10 requests arrive
       |
       v
10 concurrent executions
       |
       v
Lambda processes requests
```

If demand exceeds available concurrency, requests may be throttled or handled according to the invocation source and its retry behavior.

Important terms:

- **Reserved concurrency:** Reserves capacity for a function and sets its concurrency ceiling.
- **Provisioned concurrency:** Keeps execution environments initialized to help reduce startup latency.

These settings serve different purposes.

## 6. What happens when a limit is reached?

| Situation | Possible result |
|---|---|
| Timeout exceeded | Invocation fails |
| Concurrency exhausted | Invocation may be throttled |
| Package too large | Deployment or update fails |
| Payload too large | Invocation or integration request fails |
| Memory exhausted | Function may fail with an out-of-memory error |
| Environment variable size exceeded | Configuration update fails |

The exact behavior can depend on the integration and invocation type.

## 7. How DevOps Engineers handle limits

- Check service quotas before production deployment.
- Monitor concurrency and throttles in CloudWatch.
- Optimize function memory and execution duration.
- Use SQS to buffer workloads when suitable.
- Use pagination or batching for large workloads.
- Keep deployment packages small.
- Request a quota increase when eligible and required.
- Configure retries and dead-letter or failure destinations where appropriate.

## 8. Interview questions

**Q1. What happens if Lambda reaches its concurrency limit?**

New invocations may be throttled until capacity becomes available. The exact retry behavior depends on the invocation source.

**Q2. How can you reduce Lambda execution time?**

Optimize the code, reduce unnecessary dependencies, reuse connections where appropriate, and test suitable memory settings.

**Q3. What is reserved concurrency?**

It reserves concurrency for a function and sets the maximum number of concurrent executions for that function.

**Q4. What is provisioned concurrency?**

It prepares execution environments in advance to reduce startup latency.

**Q5. Where can you check Lambda limits?**

In the AWS Lambda quotas documentation and the Service Quotas console.

## Summary

Understanding Lambda quotas helps prevent deployment failures, throttling, performance issues, and unexpected production problems.
