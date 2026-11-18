# AWS Lambda Architecture

## 1. Introduction

AWS Lambda architecture explains how an event reaches a Lambda function, how the function executes, and how it communicates with other AWS services.

## 2. Basic architecture

```text
              Event source
          (S3 / API Gateway /
           EventBridge / SQS)
                   |
                   v
            AWS Lambda
                   |
          +--------+--------+
          |                 |
          v                 v
     AWS services       CloudWatch
   (S3 / DynamoDB /     (Logs and
    SNS / EC2 API)       Metrics)
```

## 3. Main components

### Event source

The event source starts the workflow. Examples include:

- S3 object upload.
- API Gateway HTTP request.
- EventBridge scheduled event.
- SQS message.
- SNS notification.

### Lambda function

The function contains the code that performs the task. For example, it can process a file, send a notification, or call an AWS API.

### Execution role

Lambda assumes an IAM execution role when it runs. The role gives the function permission to access resources such as S3 buckets or CloudWatch Logs.

Follow the principle of least privilege: grant only the permissions the function needs.

### Destination service

After processing an event, the function may interact with another AWS service. For example, it may save data to DynamoDB or send a message through SNS.

### Amazon CloudWatch

Lambda sends function logs to CloudWatch Logs when the required permissions and logging configuration are available. CloudWatch also provides metrics such as invocation count, errors, duration, and throttles.

## 4. Example: S3 to Lambda

Suppose a user uploads an image to an S3 bucket.

```text
User
 |
 | Upload image
 v
S3 bucket
 |
 | Object-created event
 v
Lambda function
 |
 | Process image
 v
Processed output in S3
 |
 v
CloudWatch Logs
```

Steps:

1. A user uploads an image.
2. S3 generates an object-created event.
3. The event invokes the Lambda function.
4. Lambda reads and processes the image.
5. The function saves the output.
6. Logs and execution information can be checked in CloudWatch.

Make sure the function has the required S3 permissions. Also avoid configuring an S3 trigger that causes the function to repeatedly process its own output.

## 5. Example: EventBridge scheduled automation

```text
EventBridge schedule
        |
        | Scheduled event
        v
Lambda function
        |
        | Call AWS API using Boto3
        v
Start or stop EC2 instances
        |
        v
CloudWatch Logs
```

This is a common DevOps automation pattern.

## 6. Synchronous vs asynchronous invocation

| Type | How it works | Example |
|---|---|---|
| Synchronous | Caller waits for the function response | API Gateway request |
| Asynchronous | Lambda accepts the event and processes it separately | S3 event notification |
| Poll-based | Lambda polls an event source for records | SQS queue or DynamoDB stream |

The invocation type depends on the integration and configuration.

## 7. Important architecture points

- Use IAM roles instead of storing AWS access keys in function code.
- Use CloudWatch to monitor execution and errors.
- Configure retries and failure handling based on the event source.
- Use SQS or another suitable buffering service for workloads that need decoupling.
- Design functions to handle duplicate events safely where duplicates are possible.
- Keep functions focused on one responsibility.

## 8. Interview questions

**Q1. What are the main components of Lambda architecture?**

Event source, Lambda function, execution role, destination services, and monitoring through CloudWatch.

**Q2. How does S3 invoke Lambda?**

S3 sends an event notification when a configured event occurs, such as an object upload. That notification invokes the Lambda function.

**Q3. What is the purpose of an execution role?**

It provides the AWS permissions that the Lambda function needs when it runs.

## Summary

Lambda architecture is event-driven. An event invokes a function, the function performs a task, and it can communicate with other AWS services.
