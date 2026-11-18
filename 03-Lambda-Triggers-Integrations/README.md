# AWS Lambda Triggers and Integrations

AWS Lambda can connect with other AWS services. These services can invoke a Lambda function when an event occurs or when a request is received.

This folder explains the most common Lambda integrations used by AWS DevOps Engineers.

## Folder contents

| File | Topic |
|---|---|
| [API Gateway](API-Gateway.md) | Invoke Lambda through HTTP APIs |
| [S3 Trigger](S3-Trigger.md) | Run Lambda when an S3 object event occurs |
| [EventBridge Trigger](EventBridge-Trigger.md) | Run functions for scheduled or AWS service events |
| [SNS Trigger](SNS-Trigger.md) | Process published notifications |
| [SQS Trigger](SQS-Trigger.md) | Process queued messages |
| [DynamoDB Streams](DynamoDB-Streams.md) | React to changes in a DynamoDB table |
| [CloudWatch Events](CloudWatch-Events.md) | Understand the older CloudWatch Events name and its EventBridge equivalent |
| [Step Functions](Step-Functions.md) | Coordinate multiple Lambda functions in a workflow |

## What is a Lambda trigger?

A trigger is an event source or integration that causes a Lambda function to run.

Examples:

- A user uploads a file to S3.
- A client sends an HTTP request through API Gateway.
- EventBridge sends a scheduled event.
- An application publishes a message to SNS.
- A message arrives in an SQS queue.
- A DynamoDB table record changes.

## Basic architecture

```text
        Event or request
               |
               v
     AWS service / trigger
               |
               v
        Lambda function
               |
        +------+------+
        |             |
        v             v
   Process data   Call AWS APIs
        |             |
        +------+------+
               |
               v
        CloudWatch Logs
```

## Integration types

| Integration | Typical invocation style |
|---|---|
| API Gateway | Synchronous request/response |
| S3 notifications | Asynchronous |
| EventBridge rule | Usually asynchronous |
| SNS subscription | Asynchronous |
| SQS event source mapping | Lambda polls and processes batches |
| DynamoDB Streams | Lambda polls stream records |
| Step Functions | Workflow invokes Lambda tasks |

The exact behavior, retry policy, batching, and failure handling depend on the integration configuration.

## DevOps use cases

- Process uploaded files automatically.
- Run scheduled EC2 start/stop automation.
- Send notifications when an event occurs.
- Process application messages from SQS.
- React to database changes.
- Build multi-step automation workflows.
- Connect HTTP endpoints to serverless applications.

## Important best practices

- Use IAM permissions with least privilege.
- Make event processing safe against duplicate delivery.
- Configure retries and failure handling.
- Monitor errors, duration, throttles, and queue age.
- Use dead-letter queues or failure destinations where supported and appropriate.
- Avoid recursive event configurations.
- Test integrations with realistic event payloads.

## Official documentation

- [Lambda event sources](https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html)
- [Lambda with API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)
- [Lambda with S3](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)
- [Lambda with EventBridge](https://docs.aws.amazon.com/lambda/latest/dg/with-eventbridge.html)
- [Lambda with SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
