# What is AWS Lambda?

## 1. Introduction

AWS Lambda is a serverless compute service provided by Amazon Web Services (AWS).

It allows you to run code without creating or managing servers. You upload your code, configure when it should run, and Lambda executes it when triggered by an event or request.

**Simple meaning:** You write the code. AWS manages the servers needed to run it.

## 2. How does Lambda work?

1. You create a Lambda function and upload your code.
2. You configure a trigger, such as an S3 upload or an EventBridge schedule.
3. An event occurs.
4. AWS Lambda runs your function.
5. Your function performs the required task.
6. You can view logs and metrics in Amazon CloudWatch.

### Example: Automatically process an uploaded file

```text
File uploaded to S3
        |
        v
S3 sends an event
        |
        v
Lambda function runs
        |
        v
Function processes the file
        |
        v
Logs sent to CloudWatch
```

## 3. Important Lambda terms

| Term | Simple explanation |
|---|---|
| Function | Your code and its configuration |
| Trigger | An event or service that invokes the function |
| Event | The input or information sent to the function |
| Handler | The function entry point Lambda calls |
| Runtime | The language environment used to run your code |
| Execution role | IAM role that gives the function permission to access AWS services |
| Timeout | Maximum time allowed for one invocation |
| Layer | A package for sharing libraries or other dependencies |
| Invocation | One request to run a function |

## 4. Common use cases

- Automatically create EBS snapshots.
- Start and stop EC2 instances.
- Process files uploaded to S3.
- Send notifications when an AWS event occurs.
- Run scheduled maintenance tasks.
- Respond to messages in SQS.
- Automate cloud operations using Python and Boto3.

## 5. Benefits

- No server management.
- Automatically scales with incoming requests, within service quotas.
- Pay based on requests and execution duration, subject to the pricing model.
- Integrates with many AWS services.
- Supports multiple programming languages.

## 6. Lambda vs EC2

| Feature | AWS Lambda | Amazon EC2 |
|---|---|---|
| Server management | Managed by AWS | You manage the instance |
| Execution | Runs in response to invocation | Runs while the instance is running |
| Scaling | Managed automatically | Configure scaling |
| Billing | Requests and execution duration | Instance usage |
| Best for | Event-driven tasks | Long-running applications and workloads |

## 7. DevOps example

A DevOps Engineer can use Lambda with EventBridge to stop non-production EC2 instances outside working hours.

This reduces unnecessary cloud spending.

## 8. Interview questions

**Q1. What is AWS Lambda?**

AWS Lambda is a serverless compute service that runs code in response to events without requiring us to manage servers.

**Q2. Do we need to manage servers in Lambda?**

No. AWS manages the underlying compute infrastructure.

**Q3. Can Lambda connect to other AWS services?**

Yes. It can integrate with services such as S3, SQS, SNS, EventBridge, DynamoDB, and API Gateway.

**Q4. What is a Lambda trigger?**

A trigger is an event source or service that invokes a Lambda function.

## Summary

AWS Lambda is useful for building event-driven applications and automating AWS operations without managing servers.
