# AWS Lambda Dead-Letter Queue (DLQ)

A Dead-Letter Queue (DLQ) stores failed asynchronous Lambda events so that they can be inspected and reprocessed later. Lambda supports Amazon SQS and Amazon SNS as DLQ targets for asynchronous invocation.

## DLQ flow

```text
Event Source
     |
     v
Lambda Async Queue
     |
     v
Function Invocation
     |
     +---- Success ----> Complete
     |
     +---- Repeated Failure
                 |
                 v
                DLQ
             (SQS / SNS)
                 |
                 v
         Investigate / Replay
```

A Lambda DLQ is for asynchronous invocation failures. For SQS event source mappings, configure the source queue's redrive policy and a separate dead-letter queue; that is a different mechanism.

## Create an SQS DLQ

```bash
aws sqs create-queue \
  --queue-name lambda-failure-dlq \
  --region ap-south-1
```

Retrieve the queue ARN:

```bash
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/lambda-failure-dlq \
  --attribute-names QueueArn
```

## Configure Lambda DLQ

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --dead-letter-config TargetArn=arn:aws:sqs:ap-south-1:123456789012:lambda-failure-dlq \
  --region ap-south-1
```

The Lambda execution role needs permission to send failed events to the target queue or topic. The target resource policy may also need to allow the required access.

## Read messages from the DLQ

```bash
aws sqs receive-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/lambda-failure-dlq \
  --max-number-of-messages 10 \
  --wait-time-seconds 10
```

Do not delete messages until the failure has been investigated and the recovery process is understood.

## DLQ vs asynchronous destination

Lambda asynchronous invocation also supports destinations for success and failure. Destinations can include SQS, SNS, Lambda, and EventBridge. Destinations provide invocation record information, while a DLQ stores the original event message. Choose based on the information and replay workflow you need.

## Best practices

- Configure DLQ or failure destinations for important asynchronous workloads.
- Monitor queue depth and oldest message age.
- Restrict access to failure queues because event payloads may contain sensitive data.
- Encrypt queues where required.
- Create a documented replay process and validate events before replaying.
- Make handlers idempotent to prevent duplicate side effects.
- Set retention long enough for investigation, but not indefinitely without a reason.
