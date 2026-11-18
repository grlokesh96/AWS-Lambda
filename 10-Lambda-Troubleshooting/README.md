# AWS Lambda Troubleshooting

This folder provides a practical troubleshooting guide for common AWS Lambda failures, including timeouts, IAM permission errors, memory exhaustion, throttling, VPC connectivity, trigger failures, and CloudWatch debugging.

## Contents

| File | What you will learn |
|---|---|
| [Timeout-Errors.md](Timeout-Errors.md) | Diagnose and fix Lambda timeout errors |
| [Permission-Denied.md](Permission-Denied.md) | Troubleshoot IAM and resource-policy failures |
| [Memory-Exceeded.md](Memory-Exceeded.md) | Investigate memory exhaustion and optimize memory |
| [Throttling-Errors.md](Throttling-Errors.md) | Identify concurrency and quota throttling |
| [VPC-Connectivity.md](VPC-Connectivity.md) | Debug networking and private resource access |
| [Trigger-Failures.md](Trigger-Failures.md) | Troubleshoot event source and invocation failures |
| [CloudWatch-Debugging.md](CloudWatch-Debugging.md) | Use logs, metrics, and Logs Insights to find root causes |

## Troubleshooting workflow

```text
Incident / Alert
      |
      v
Check Lambda State and Recent Deployments
      |
      v
Review CloudWatch Metrics
      |
      v
Find Invocation Logs and Request ID
      |
      v
Check IAM / Network / Runtime / Trigger
      |
      v
Identify Root Cause
      |
      v
Fix in Test Environment
      |
      v
Deploy and Verify
      |
      v
Update Runbook and Add Monitoring
```

## First commands to run

```bash
# Function configuration
aws lambda get-function-configuration \
  --function-name demo-lambda

# Recent function logs
aws logs tail /aws/lambda/demo-lambda \
  --since 1h

# Function concurrency
aws lambda get-function-concurrency \
  --function-name demo-lambda

# Event source mappings
aws lambda list-event-source-mappings \
  --function-name demo-lambda
```

## Incident checklist

- [ ] Identify affected function, Region, and time window.
- [ ] Check whether a deployment or configuration change happened.
- [ ] Review `Errors`, `Duration`, `Throttles`, and concurrency metrics.
- [ ] Find the failed invocation's request ID and stack trace.
- [ ] Check execution role, resource policy, and KMS permissions.
- [ ] Check VPC routes, security groups, DNS, and endpoints if relevant.
- [ ] Check trigger configuration, retries, and event age.
- [ ] Verify the fix with a representative test event.
- [ ] Document root cause and preventive action.

> Avoid changing several settings at once during an incident. Make a controlled change, verify the result, and preserve enough logs and metrics for root-cause analysis.
