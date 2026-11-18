# AWS Lambda with Amazon SNS

## 1. Introduction

Amazon SNS (Simple Notification Service) is a messaging service used to publish messages and deliver them to subscribers.

A Lambda function can subscribe to an SNS topic. When a message is published, SNS can invoke the function.

## 2. Architecture

```text
Application / Publisher
          |
          | Publish message
          v
       SNS Topic
          |
          | Notification
          v
    Lambda Function
          |
          v
 Process notification
          |
          v
    CloudWatch Logs
```

## 3. How it works

1. Create an SNS topic.
2. Subscribe a Lambda function to the topic.
3. A publisher sends a message.
4. SNS delivers the message to the Lambda subscription.
5. Lambda processes the message.

## 4. Common use cases

- Send alerts when an application event occurs.
- Process notifications.
- Trigger automated remediation.
- Notify teams about infrastructure events.
- Fan out one message to multiple subscribers.

## 5. Create an SNS topic

```bash
aws sns create-topic \
  --name devops-alerts
```

Save the topic ARN from the response.

## 6. Subscribe Lambda to SNS

```bash
aws sns subscribe \
  --topic-arn <SNS_TOPIC_ARN> \
  --protocol lambda \
  --notification-endpoint <LAMBDA_FUNCTION_ARN>
```

Replace both placeholders with actual ARNs.

The Lambda function must allow SNS to invoke it. When using the Console, the required permission can be configured during subscription setup. For CLI-based setup, add the resource permission if it is not already present.

Example:

```bash
aws lambda add-permission \
  --function-name my-lambda-function \
  --statement-id AllowSNSInvoke \
  --action lambda:InvokeFunction \
  --principal sns.amazonaws.com \
  --source-arn <SNS_TOPIC_ARN>
```

## 7. Publish a message

```bash
aws sns publish \
  --topic-arn <SNS_TOPIC_ARN> \
  --message "Production deployment completed successfully"
```

## 8. Example Python Lambda function

```python
def lambda_handler(event, context):
    for record in event.get("Records", []):
        message = record["Sns"]["Message"]
        subject = record["Sns"].get("Subject")

        print("Subject:", subject)
        print("Message:", message)

    return {"statusCode": 200}
```

SNS event records contain message details inside the `Sns` object.

## 9. SNS vs SQS

| SNS | SQS |
|---|---|
| Publish/subscribe messaging | Message queue |
| Sends messages to subscribers | Stores messages until consumed |
| Useful for fan-out | Useful for buffering and decoupling |
| Subscribers receive notifications | Consumers poll or use event source mappings |

SNS can publish to multiple subscribers. An SNS topic can also deliver messages to an SQS queue.

## 10. Best practices

- Restrict who can publish to the topic.
- Use encryption where required.
- Make message processing safe against duplicates.
- Monitor delivery failures.
- Configure appropriate retry and failure handling.
- Avoid putting secrets in notification messages.

## 11. Interview questions

**Q1. What is SNS?**

SNS is a managed publish/subscribe messaging service.

**Q2. How does SNS invoke Lambda?**

The Lambda function subscribes to an SNS topic. SNS invokes it when a message is published.

**Q3. Can one SNS topic have multiple subscribers?**

Yes. It can deliver messages to multiple supported subscriber types.

## Summary

SNS and Lambda are useful for notification-driven automation and event fan-out.
