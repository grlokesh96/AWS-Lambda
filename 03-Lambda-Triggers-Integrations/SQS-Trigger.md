# AWS Lambda with Amazon SQS

## 1. Introduction

Amazon SQS (Simple Queue Service) is a managed message queue.

Applications send messages to a queue. Lambda can poll the queue through an event source mapping and process messages in batches.

**Simple example:** An application sends a job to SQS. Lambda reads the message and processes the job.

## 2. Architecture

```text
Application
     |
     | Send message
     v
  SQS Queue
     |
     | Lambda polls queue
     v
Lambda Function
     |
     | Process message
     v
Delete successfully processed
messages from queue
```

Lambda manages polling for an SQS event source mapping. It invokes the function with batches of messages.

## 3. How it works

1. An application sends a message to SQS.
2. The message is stored in the queue.
3. Lambda polls the queue.
4. Lambda receives a batch of messages.
5. The function processes the batch.
6. Successfully processed messages are removed according to the event source mapping behavior.

## 4. Common use cases

- Background job processing.
- Order processing.
- Log processing.
- Decoupling microservices.
- Handling traffic spikes.
- Retrying work after temporary failures.

## 5. Create an SQS queue

```bash
aws sqs create-queue \
  --queue-name devops-task-queue
```

Get the queue URL:

```bash
aws sqs get-queue-url \
  --queue-name devops-task-queue
```

Get the queue ARN:

```bash
aws sqs get-queue-attributes \
  --queue-url <QUEUE_URL> \
  --attribute-names QueueArn
```

## 6. Example Python Lambda function

```python
def lambda_handler(event, context):
    for record in event.get("Records", []):
        message = record["body"]
        print("Processing message:", message)

    return {"statusCode": 200}
```

This basic example processes each record in the received batch. Production code should handle failures carefully.

## 7. Configure the SQS trigger

Using the AWS Console:

1. Open Lambda.
2. Select your function.
3. Choose **Add trigger**.
4. Select SQS.
5. Select the queue.
6. Configure batch size and other options.
7. Add the trigger.

The Lambda execution role needs the required SQS polling permissions. The Console can help add the standard permissions.

## 8. Important SQS settings

| Setting | Purpose |
|---|---|
| Visibility timeout | Hides a received message temporarily while it is being processed |
| Message retention | Controls how long messages remain in the queue |
| Batch size | Maximum number of records sent in one invocation, within service limits |
| Dead-letter queue | Stores messages that could not be processed successfully after configured receives |
| Event source mapping | Connects the queue to the Lambda function |

For SQS-triggered Lambda functions, configure the queue visibility timeout appropriately. AWS recommends it be at least six times the function timeout, plus the maximum batching window, when using the standard recommendation.

## 9. Handling failed messages

If a batch fails, messages can become visible again after the visibility timeout and be retried.

For more precise failure handling, use partial batch responses. This allows the function to report only the records that failed, rather than causing successfully processed records in the batch to be retried.

Configure a redrive policy and dead-letter queue for messages that repeatedly fail.

## 10. Standard queue vs FIFO queue

| Standard queue | FIFO queue |
|---|---|
| High throughput | Ordered processing within message groups |
| At-least-once delivery | Exactly-once processing features with deduplication configuration |
| Duplicates can occur | Deduplication helps prevent duplicate sends within its deduplication scope |

Applications should still be designed to handle retries and duplicate processing safely.

## 11. Interview questions

**Q1. What is SQS?**

SQS is a managed message queue used to decouple applications and process messages asynchronously.

**Q2. How does Lambda consume SQS messages?**

Lambda polls the queue through an event source mapping and invokes the function with message batches.

**Q3. What is a dead-letter queue?**

It is a queue used to hold messages that could not be processed successfully after the configured number of receives.

**Q4. What is visibility timeout?**

It is the period during which a received message is hidden from other consumers while it is being processed.

## Summary

SQS with Lambda is a common pattern for reliable background processing and decoupling applications.
