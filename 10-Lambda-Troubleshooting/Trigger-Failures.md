# AWS Lambda Trigger Failures

A Lambda trigger failure occurs when an event source cannot invoke the function, an event source mapping cannot process records, or the function fails while handling the event.

## Common Lambda triggers

- API Gateway
- Amazon S3
- Amazon EventBridge
- Amazon SNS
- Amazon SQS
- DynamoDB Streams
- Kinesis Data Streams

Each integration has its own permissions and delivery behavior.

## Step 1: Identify the trigger type

Determine whether the integration uses:

- Direct synchronous invocation.
- Asynchronous invocation.
- An event source mapping that polls a queue or stream.

This determines where to look for retries and failed events.

## Step 2: Check Lambda resource policy

For service-based invocation, verify that the Lambda resource policy allows the service principal and is scoped to the expected source ARN/account.

```bash
aws lambda get-policy \
  --function-name demo-lambda
```

## Step 3: Check event source mappings

```bash
aws lambda list-event-source-mappings \
  --function-name demo-lambda
```

Inspect a mapping:

```bash
aws lambda get-event-source-mapping \
  --uuid <event-source-mapping-uuid>
```

Check state, last processing result, batch size, and source configuration.

## Step 4: Check source-specific behavior

| Trigger | What to inspect |
|---|---|
| API Gateway | Integration URI, invoke permission, payload format, stage logs |
| S3 | Bucket notification, Lambda permission, Region alignment, recursive trigger loops |
| EventBridge | Rule state, event pattern, target, invoke permission, delivery metrics |
| SNS | Subscription, topic policy, Lambda invoke permission |
| SQS | Queue ARN, execution role, visibility timeout, redrive policy, mapping state |
| DynamoDB Streams | Stream enabled, mapping state, execution role, iterator age |
| Kinesis | Stream status, shard/record processing, iterator age, mapping configuration |

## Step 5: Test the function directly

```bash
aws lambda invoke \
  --function-name demo-lambda \
  --payload '{"test":true}' \
  response.json

cat response.json
```

A direct test validates the handler path but does not prove the trigger configuration is correct.

## Best practices

- Test the actual event source, not only a manually constructed event.
- Keep permissions narrowly scoped.
- Monitor event age, queue depth, failed invocations, and mapping state.
- Configure retries and failure destinations based on the trigger type.
- Make processing idempotent.
- Avoid S3 notification loops where the function writes back into the same triggering bucket/prefix.
