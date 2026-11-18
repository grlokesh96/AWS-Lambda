# AWS Lambda Provisioned Concurrency

Provisioned Concurrency keeps a configured number of Lambda execution environments initialized and ready to respond. It helps reduce initialization latency for latency-sensitive workloads.

## How it works

```text
Incoming Requests
       |
       v
  Lambda Alias
       |
       v
Provisioned Environments
  [Ready] [Ready] [Ready]
       |
       v
  Function Handler
```

Provisioned Concurrency is configured for a published version or an alias that points to a published version. It is not configured directly for `$LATEST`.

## When to use it

- APIs with strict latency objectives.
- Predictable scheduled traffic.
- Workloads with known peaks.
- Applications where cold-start latency is noticeable.

## Create an alias

```bash
aws lambda publish-version \
  --function-name demo-lambda \
  --description "Release for provisioned concurrency"
```

Assume the published version is `5`:

```bash
aws lambda create-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 5
```

## Configure provisioned concurrency

```bash
aws lambda put-provisioned-concurrency-config \
  --function-name demo-lambda \
  --qualifier prod \
  --provisioned-concurrent-executions 5
```

Check the configuration:

```bash
aws lambda get-provisioned-concurrency-config \
  --function-name demo-lambda \
  --qualifier prod
```

Wait until the status is ready before relying on the configured capacity.

## Important considerations

- Provisioned Concurrency incurs additional charges while configured.
- It does not mean the function can never scale beyond the provisioned amount; requests above it may use on-demand capacity, subject to concurrency limits and behavior.
- Configure enough capacity for expected traffic and test bursts.
- Use Application Auto Scaling where appropriate to adjust provisioned concurrency based on demand.
- Account for deployment changes: publish a new version, update the alias, and configure provisioned capacity for the target qualifier as needed.

## Best practices

- Use it only for latency-sensitive functions.
- Start with measured concurrency demand.
- Monitor provisioned concurrency utilization and spillover.
- Schedule or autoscale capacity for predictable traffic patterns.
- Include the extra cost in the performance decision.
