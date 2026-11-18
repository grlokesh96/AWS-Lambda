# AWS Lambda Throttling Errors

Lambda throttles an invocation when the function cannot accept more concurrent executions because of concurrency limits or other capacity constraints.

## Common symptoms

- `TooManyRequestsException`
- Throttle counts increase in CloudWatch.
- API callers receive throttling or retryable errors.
- Event queues or stream records begin to age.

## Common causes

- Account concurrency quota reached.
- Function reserved concurrency is too low.
- Sudden traffic spike.
- Provisioned concurrency capacity is insufficient for the expected low-latency traffic.
- Downstream systems require a lower concurrency limit.

## Step 1: Check function concurrency

```bash
aws lambda get-function-concurrency \
  --function-name demo-lambda
```

Check account-level concurrency:

```bash
aws lambda get-account-settings
```

## Step 2: Check CloudWatch metrics

Review:

- `Throttles`
- `ConcurrentExecutions`
- `UnreservedConcurrentExecutions`
- `ProvisionedConcurrencyUtilization`, where applicable
- Event source age metrics, where applicable

## Step 3: Review reserved concurrency

Reserved concurrency sets a function's maximum concurrency and reserves that amount from the account pool. If the limit is too low, the function may throttle even when the account has capacity.

Example:

```bash
aws lambda put-function-concurrency \
  --function-name demo-lambda \
  --reserved-concurrent-executions 50
```

Choose a value based on account capacity and downstream service limits.

## Step 4: Review event source configuration

For SQS, review batch size, maximum concurrency, queue visibility timeout, and redrive policy. For stream sources, review shard count, parallelization, and iterator age.

## Step 5: Use retries safely

Synchronous callers should implement bounded retries with exponential backoff and jitter. Asynchronous invocations and event source mappings have their own retry behavior. Make handlers idempotent to handle duplicate delivery.

## Best practices

- Set concurrency based on downstream capacity.
- Monitor throttles and event age.
- Request a quota increase only after checking workload needs and account limits.
- Use reserved concurrency to protect critical workloads or cap pressure.
- Use provisioned concurrency for startup latency, not as a general fix for every throttle.
- Apply backpressure and queue-based buffering for bursty workloads.
