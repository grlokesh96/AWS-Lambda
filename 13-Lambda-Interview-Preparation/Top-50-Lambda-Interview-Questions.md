# Top 50 AWS Lambda Interview Questions and Answers

## Fundamentals

### 1. What is AWS Lambda?
AWS Lambda is a serverless compute service that runs code in response to events. AWS manages the underlying infrastructure, while you configure the function, runtime, permissions, memory, timeout, and scaling behavior.

### 2. What is serverless computing?
Serverless means the cloud provider manages server provisioning and much of the operational infrastructure. It does not mean servers do not exist. Customers remain responsible for application code, permissions, configuration, and reliability.

### 3. What is a Lambda function?
A Lambda function is the deployable unit of code and configuration that AWS invokes in response to an event or direct invocation.

### 4. What is a Lambda handler?
The handler is the function entry point called by the Lambda runtime. In Python, `app.lambda_handler` means the file is `app.py` and the function is `lambda_handler`.

### 5. What is the Lambda execution role?
It is an IAM role assumed by the Lambda service. Its permissions determine which AWS APIs the function code can call.

### 6. What is the difference between execution role and resource-based policy?
The execution role grants the function permission to access AWS resources. A resource-based policy grants another principal or service permission to invoke or access a resource.

### 7. What is a Lambda deployment package?
It contains the function code and, when needed, dependencies. A function can be deployed as a ZIP package or as a container image stored in ECR.

### 8. What are Lambda layers?
Layers package reusable libraries, custom runtimes, or other dependencies separately from function code. A function can use configured layers at runtime.

### 9. What runtimes does Lambda support?
Lambda supports managed runtimes for several languages, plus custom runtimes and container images. The supported runtime list changes; check AWS documentation before choosing one.

### 10. What is the Lambda execution environment?
It is the isolated environment in which Lambda initializes the runtime and executes function invocations. AWS may reuse an environment for later invocations, but reuse is not guaranteed.

## Invocation and event sources

### 11. What are the main invocation types?
The main invocation types are synchronous (`RequestResponse`), asynchronous (`Event`), and poll-based event source mappings for services such as SQS and Kinesis.

### 12. What is the difference between synchronous and asynchronous invocation?
With synchronous invocation, the caller waits for the function response. With asynchronous invocation, Lambda accepts the event and processes it later; the caller does not wait for the function result.

### 13. What is an event source mapping?
It is a Lambda-managed poller configuration that reads records from supported event sources such as SQS, Kinesis, and DynamoDB Streams, then invokes the function with batches.

### 14. How can S3 trigger Lambda?
Configure an S3 event notification for events such as object creation and grant S3 permission to invoke the function. Use prefix/suffix filters to select objects.

### 15. How does API Gateway integrate with Lambda?
API Gateway receives an HTTP request and invokes Lambda through an integration. Lambda returns a response in the expected proxy or configured integration format.

### 16. How does SQS integrate with Lambda?
An event source mapping polls the queue and invokes Lambda with batches of messages. Failed messages may become visible again after the visibility timeout, and a queue redrive policy can move repeatedly failed messages to a DLQ.

### 17. How does EventBridge invoke Lambda?
An EventBridge rule or schedule matches an event and targets a Lambda function. The function's resource-based policy must allow EventBridge invocation.

### 18. What is the difference between SNS and SQS triggers?
SNS pushes notifications to subscribed endpoints, including Lambda. SQS is a durable queue that Lambda polls through an event source mapping, allowing buffering and consumer-based processing.

### 19. Can one Lambda function have multiple triggers?
Yes. Multiple services or event source mappings can invoke the same function, but the handler must correctly identify and process each event schema.

### 20. What is a Lambda destination?
A destination routes the result of an asynchronous invocation to a target such as SQS, SNS, EventBridge, or another Lambda. Destinations can include invocation records and are different from stream event source mapping failure destinations.

## Performance and scaling

### 21. What is a cold start?
A cold start occurs when Lambda must create or initialize an execution environment before running the handler. Initialization may include runtime startup and loading dependencies.

### 22. How do you reduce cold starts?
Keep deployment packages small, minimize initialization work, reuse SDK clients outside the handler, choose suitable runtimes, and consider provisioned concurrency for latency-sensitive functions.

### 23. What is provisioned concurrency?
It pre-initializes a configured number of execution environments so they are ready to respond with lower initialization latency. It incurs additional cost.

### 24. What is reserved concurrency?
It reserves concurrency capacity for a function and also sets its maximum concurrent executions. It can protect downstream systems but may throttle excess invocations.

### 25. What is account concurrency?
It is the concurrency capacity available to functions in an AWS account and Region, subject to service quotas and any applicable allocation rules.

### 26. How does Lambda scale?
Lambda creates execution environments in response to incoming demand, subject to concurrency quotas, scaling behavior, event source limits, and downstream capacity.

### 27. How do you optimize Lambda memory?
Benchmark realistic workloads at different memory settings. More memory also provides more CPU allocation, so a higher setting can sometimes reduce execution time and total cost.

### 28. What is the maximum execution duration?
The maximum configured timeout is 15 minutes. Choose a timeout based on the expected work and downstream timeouts rather than setting the maximum by default.

### 29. What is ephemeral storage?
It is temporary `/tmp` storage available to a Lambda execution environment. Configure its size according to workload needs; do not treat it as durable storage.

### 30. How do you handle large files?
Avoid loading an entire large object into memory. Stream data, process in chunks, or use services such as S3, SQS, Step Functions, Glue, or ECS depending on the workload.

## Security and networking

### 31. How do you secure a Lambda function?
Use least-privilege IAM, secure secrets, encrypt data, validate inputs, patch dependencies, restrict invocation permissions, and monitor activity.

### 32. How do you access Secrets Manager from Lambda?
Grant `secretsmanager:GetSecretValue` for the specific secret and retrieve it using the AWS SDK. Consider caching the value between warm invocations while respecting rotation requirements.

### 33. Why place Lambda in a VPC?
Place it in a VPC when it must access private resources such as internal databases or services. VPC attachment alone does not automatically provide internet access.

### 34. How does Lambda access the internet from a private subnet?
A typical IPv4 design routes outbound traffic through a NAT Gateway in a public subnet. Also verify route tables, security groups, network ACLs, DNS, and destination policies.

### 35. What is the difference between security groups and IAM policies?
IAM controls which AWS API actions a principal can perform. Security groups control network traffic to and from supported network interfaces and resources.

### 36. How do you encrypt Lambda environment variables?
Lambda encrypts environment variables at rest using AWS-managed encryption by default; you can configure a customer managed KMS key for additional control. Restrict who can read or update function configuration.

### 37. How do you restrict who can invoke Lambda?
Use the function's resource-based policy, identity policies, and service-specific controls. For API endpoints, also configure API Gateway authorization and network protections as required.

## Reliability and troubleshooting

### 38. What happens when Lambda times out?
The invocation is terminated when its configured timeout is reached. Review duration, logs, downstream latency, memory, and the function's timeout configuration.

### 39. What causes Lambda throttling?
Throttling can occur when the function reaches its concurrency limit or account capacity, or when event source and service-specific limits apply.

### 40. How do retries work?
Retry behavior depends on the invocation model and event source. Asynchronous invocations have Lambda-managed retry behavior; SQS and stream sources use event source mapping behavior and their own retry/visibility or checkpoint semantics.

### 41. What is a dead-letter queue?
A DLQ is a destination, commonly SQS or SNS, for events that could not be processed after the configured asynchronous retry policy. For stream and SQS event source mappings, configure the relevant failure handling such as an on-failure destination or queue redrive policy.

### 42. What is idempotency?
Idempotency means processing the same logical event more than once has the same final effect as processing it once. Use a stable event or business key and a durable deduplication mechanism.

### 43. How do you monitor Lambda?
Use CloudWatch metrics, CloudWatch Logs, alarms, tracing with AWS X-Ray or supported tracing integrations, and structured application logs.

### 44. What are important Lambda metrics?
Common metrics include `Invocations`, `Errors`, `Duration`, `Throttles`, `ConcurrentExecutions`, `AsyncEventAge`, and `IteratorAge` for relevant stream workloads.

### 45. How do you troubleshoot AccessDenied errors?
Identify the denied API action and resource from the error and CloudTrail where available. Check the execution role, resource policy, permission boundary, SCP, KMS key policy, and explicit denies.

## Deployment and DevOps

### 46. What is Lambda versioning?
Publishing a version creates an immutable snapshot of function code and most configuration. `$LATEST` remains the mutable working version.

### 47. What is a Lambda alias?
An alias is a named pointer to a published version, such as `dev`, `staging`, or `prod`. It can support weighted traffic shifting between versions.

### 48. How do you implement blue-green deployment?
Publish a new version, point an alias to it, shift a controlled percentage of traffic, monitor alarms and application health, and roll back the alias if necessary.

### 49. How do you deploy Lambda using CI/CD?
Use a pipeline such as GitHub Actions, Jenkins, GitLab CI, or AWS CodePipeline. Build and test the artifact, scan dependencies, assume an AWS role using OIDC where supported, deploy with SAM/CloudFormation/Terraform, and verify the release.

### 50. How do you roll back a Lambda deployment?
Move the production alias back to the previous known-good version or use the deployment platform's rollback mechanism. Confirm that environment variables, layers, permissions, and dependent resources remain compatible.
