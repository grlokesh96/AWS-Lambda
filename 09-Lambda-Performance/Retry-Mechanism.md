# AWS Lambda Retry Mechanisms

Lambda retry behavior depends on how the function is invoked. Synchronous invocation, asynchronous invocation, and event source mappings have different retry and failure-handling models.

## Invocation types

| Invocation type | Example | Retry behavior |
|---|---|---|
| Synchronous | API Gateway or direct `RequestResponse` call | The caller receives the result or error and decides whether to retry |
| Asynchronous | S3 or EventBridge invoking Lambda asynchronously | Lambda manages retries for function errors according to asynchronous invocation settings |
| Event source mapping | SQS, Kinesis, DynamoDB Streams | The event source mapping polls records and handles retries according to its source-specific configuration |

Do not assume every Lambda trigger uses the same retry policy.

## Synchronous invocation

For synchronous calls, the caller receives the error response. The caller or upstream service decides whether and when to retry. Use bounded retries and exponential backoff where appropriate.

## Asynchronous invocation

For asynchronous invocation, Lambda queues the event and retries function errors according to the configured retry policy. You can configure maximum event age, retry attempts, and a failure destination or dead-letter queue.

Example:

```bash
aws lambda put-function-event-invoke-config \
  --function-name demo-lambda \
  --maximum-retry-attempts 1 \
  --maximum-event-age-in-seconds 3600
```

Inspect the configuration:

```bash
aws lambda get-function-event-invoke-config \
  --function-name demo-lambda
```

These settings apply to asynchronous invocation, not all event sources.

## SQS event source mapping

For SQS, configure visibility timeout, batch size, and a redrive policy on the queue. Lambda polls the queue and deletes messages after successful processing. Failed messages can become visible again after the visibility timeout and may be retried.

For batch processing, consider partial batch responses so successfully processed records do not need to be retried with failed records.

## Idempotency

Retries can cause the same event to be processed more than once. Design handlers to be idempotent.

Common techniques:

- Use a stable event or business transaction ID.
- Store processed IDs with conditional writes.
- Make updates safe to repeat.
- Avoid non-idempotent side effects without deduplication.

## Best practices

- Understand the trigger's invocation model.
- Use bounded retries and backoff.
- Configure DLQs or destinations for asynchronous failures.
- Configure SQS redrive policies for queue-based workloads.
- Monitor event age, receive counts, and failure rates.
- Test duplicate delivery and partial failure scenarios.
