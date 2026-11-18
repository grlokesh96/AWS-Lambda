# AWS Lambda Scenario-Based Interview Questions

Use these scenarios to practice explaining a production problem, diagnosis, solution, and verification.

## 1. Lambda is timing out while calling RDS

**Likely causes**
- Database connection is slow or unavailable.
- Lambda timeout is too low.
- Security group or subnet routing is incorrect.
- Too many concurrent connections exhaust database capacity.

**Approach**
1. Check CloudWatch duration and error logs.
2. Verify Lambda subnet routes and security group egress.
3. Verify the RDS security group allows traffic from the Lambda security group on the database port.
4. Review connection reuse and connection limits.
5. Use RDS Proxy where appropriate and tune concurrency.

**Interview answer:** I would first separate network connectivity from database latency using logs and metrics. Then I would validate VPC routing and security groups, review connection counts, and introduce connection pooling or RDS Proxy if connection churn is the issue.

## 2. Lambda receives duplicate S3 events

S3 event notifications are at-least-once and can be delivered more than once. Make processing idempotent using the bucket, object key, version ID, or event identity as a deduplication key. Store completed processing state in DynamoDB or another durable store.

## 3. Lambda is throttled during a traffic spike

Check `Throttles`, `ConcurrentExecutions`, account concurrency, and downstream capacity. Consider reserved concurrency, provisioned concurrency for latency-sensitive traffic, SQS buffering, and controlled producer rates. Do not simply increase concurrency if the database or API cannot handle it.

## 4. Lambda cannot access the internet after VPC attachment

A Lambda function in a private subnet needs a valid route to a NAT Gateway for typical IPv4 internet egress. Check subnet route tables, NAT health, security groups, network ACLs, DNS, and any egress firewall. A public IP is not automatically assigned to a Lambda ENI.

## 5. Lambda returns AccessDenied when reading an S3 object

Check the function execution role's `s3:GetObject` permission, bucket policy, object ARN and prefix, account boundaries, and any explicit deny. If the object uses SSE-KMS, also verify `kms:Decrypt` and the KMS key policy.

## 6. SQS messages are processed repeatedly

Check function errors, visibility timeout, batch size, and processing duration. The queue visibility timeout should be configured in relation to the function timeout and retry strategy. Make the handler idempotent and configure a redrive policy with a DLQ.

## 7. One bad record causes a whole batch to fail

For supported event source mappings, use partial batch response reporting (for example, `ReportBatchItemFailures`) so successfully processed records do not need to be retried with the failed record. Configure it correctly for the source type and handler response format.

## 8. Lambda cold-start latency affects an API

Measure initialization and invocation latency separately. Reduce package size and initialization work, reuse clients, and consider provisioned concurrency for predictable low latency. Verify the impact and cost under realistic traffic.

## 9. A deployment introduces errors in production

Use a Lambda alias that points to published versions. Shift traffic gradually, monitor errors, duration, and business metrics, then move the alias back to the previous version if alarms breach. Preserve deployment artifacts and release metadata.

## 10. Lambda logs are missing

Verify the execution role includes CloudWatch Logs permissions, the correct Region and log group, and that the function is actually invoked. Check whether logging is configured and whether the runtime or application fails before expected log statements.

## 11. A function's duration suddenly increases

Compare recent deployments, input size, dependency latency, downstream throttling, cold starts, and memory settings. Use structured logs and tracing to identify the slow segment before changing memory or timeout.

## 12. An EC2 automation Lambda stops the wrong instances

Immediately disable the schedule or function trigger and assess impact. Improve selection using explicit allow-list tags, account/Region checks, environment exclusions, dry-run mode, and approval for production. Log every selected instance before action.

## 13. An ECR cleanup function deletes an image needed for rollback

Stop cleanup, review CloudTrail and deployment records, and restore the image if a copy or replication exists. Change the policy to preserve deployed digests, protected tags, and rollback windows. Prefer native ECR lifecycle policies when they meet requirements.

## 14. Lambda processes a file and triggers itself forever

The function likely writes output to a location included by the same S3 notification. Use separate buckets or event prefix/suffix filters so output objects do not match the input trigger.

## 15. Lambda cost increases unexpectedly

Review invocation count, duration, memory, provisioned concurrency, retries, and event volume. Check for recursive triggers, duplicate processing, large payloads, or excessive polling. Optimize only after identifying the cost driver.

## 16. EventBridge invokes the function but the handler fails

Inspect the exact event JSON and compare it with the handler's expected schema. Event formats differ by source. Add defensive parsing, validate required fields, and test with captured representative events.

## 17. Lambda needs to process a job longer than 15 minutes

Do not try to bypass the Lambda maximum timeout. Split the job into smaller tasks and orchestrate with Step Functions, SQS, or EventBridge, or use ECS/Fargate, AWS Batch, or another compute service suited to long-running workloads.

## 18. A secret rotates but Lambda keeps using the old value

Review whether the function caches the secret in a warm execution environment. Add a cache TTL or refresh strategy aligned with rotation, and ensure the function can retrieve the new version. Avoid logging secret values.

## 19. Lambda has too many database connections

Reuse connections where appropriate, limit concurrency, use RDS Proxy, and ensure connection cleanup. Estimate maximum connections from concurrency multiplied by connections per execution environment.

## 20. A security finding triggers an unsafe automatic change

Switch remediation to alert-only mode, validate event matching and resource scope, and introduce allow-lists, approval gates, dry-run, and rollback. Detection and remediation should use separate IAM roles.

## 21. How would you design a resilient file-processing system?

Use S3 for object storage, EventBridge or SQS for buffering/routing, Lambda for stateless processing, DynamoDB for idempotency state, and a DLQ or failure destination for poison events. Monitor age, retries, errors, and throughput.

## 22. How would you design multi-account Lambda automation?

Use a central orchestration account or delegated administration where appropriate. Assume narrowly scoped roles in target accounts, restrict trust policies to the central role, record account/Region context, and handle partial failures independently.

## 23. How would you protect Lambda deployment credentials?

Prefer GitHub Actions or another CI platform's OIDC federation to assume an IAM role. Restrict the trust policy to the exact repository and branch/environment. Avoid long-lived access keys.

## 24. How would you reduce alert noise?

Deduplicate events, group related alerts, define severity and suppression windows, include runbook links, and alert on actionable symptoms rather than every state transition. Keep an audit trail of suppressed alerts.

## 25. When would you not choose Lambda?

Avoid Lambda when workloads need sustained execution beyond its timeout, specialized host control, persistent local state, very high predictable throughput that is cheaper elsewhere, or compute patterns better suited to containers or batch services. Choose based on workload characteristics and total operational cost.
