# AWS Lambda DevOps Cheat Sheet

Quick reference for daily operations, deployment, troubleshooting, and interviews.

## AWS CLI

Set common variables:

```bash
export AWS_REGION=ap-south-1
export FUNCTION_NAME=my-lambda-function
```

List functions:

```bash
aws lambda list-functions \
  --region "$AWS_REGION" \
  --query 'Functions[].FunctionName'
```

Get function configuration:

```bash
aws lambda get-function-configuration \
  --function-name "$FUNCTION_NAME" \
  --region "$AWS_REGION"
```

Invoke a function:

```bash
aws lambda invoke \
  --function-name "$FUNCTION_NAME" \
  --payload '{"hello":"world"}' \
  --cli-binary-format raw-in-base64-out \
  response.json
cat response.json
```

Tail logs:

```bash
aws logs tail "/aws/lambda/$FUNCTION_NAME" \
  --since 1h \
  --follow \
  --region "$AWS_REGION"
```

Update function code from a ZIP package:

```bash
zip -r function.zip .
aws lambda update-function-code \
  --function-name "$FUNCTION_NAME" \
  --zip-file fileb://function.zip \
  --region "$AWS_REGION"
```

Publish a version:

```bash
aws lambda publish-version \
  --function-name "$FUNCTION_NAME" \
  --region "$AWS_REGION"
```

Update an alias:

```bash
aws lambda update-alias \
  --function-name "$FUNCTION_NAME" \
  --name prod \
  --function-version 3 \
  --region "$AWS_REGION"
```

Check concurrency:

```bash
aws lambda get-function-concurrency \
  --function-name "$FUNCTION_NAME" \
  --region "$AWS_REGION"
```

## Common Lambda metrics

| Metric | What it indicates |
|---|---|
| `Invocations` | Number of function invocations |
| `Errors` | Invocations that failed |
| `Duration` | Execution duration |
| `Throttles` | Invocations rejected due to concurrency limits |
| `ConcurrentExecutions` | Concurrent executions |
| `AsyncEventAge` | Age of asynchronous events waiting/being processed |
| `IteratorAge` | Age of the last record in stream processing |

Metric availability depends on the function and invocation model.

## Common errors and first checks

| Symptom | First checks |
|---|---|
| `Task timed out` | Timeout, downstream latency, network, memory |
| `AccessDeniedException` | Execution role, resource policy, SCP, KMS key policy |
| `TooManyRequestsException` | Reserved/account concurrency, event source limits |
| `ImportModuleError` | ZIP folder structure, dependency packaging, runtime |
| VPC connection timeout | Subnets, routes, security groups, NACL, DNS |
| SQS messages repeat | Function errors, visibility timeout, DLQ, idempotency |
| S3 trigger loop | Input/output prefix and notification filters |
| No logs | Log permissions, correct Region/log group, invocation |

## Event source reminders

| Source | Integration model | Reliability consideration |
|---|---|---|
| S3 | Service event notification | At-least-once; duplicate/out-of-order events |
| EventBridge | Rule or schedule target | Retry policy and DLQ for target delivery |
| SQS | Event source mapping poller | Visibility timeout, batch failure, redrive |
| Kinesis | Event source mapping | Checkpointing, shard concurrency, iterator age |
| DynamoDB Streams | Event source mapping | Checkpointing and partial batch failures |
| API Gateway | Synchronous integration | Caller receives response; validate request |

## Security checklist

- [ ] Dedicated least-privilege execution role.
- [ ] Resource-based invoke policy restricted to expected source.
- [ ] Secrets stored outside source code.
- [ ] KMS permissions and key policy reviewed.
- [ ] Dependencies scanned and updated.
- [ ] Inputs validated and logs scrubbed of sensitive data.
- [ ] CI/CD uses OIDC or managed identity federation.
- [ ] Production changes use reviewed approvals.

## Deployment checklist

- [ ] Unit and integration tests pass.
- [ ] Linting and dependency/security scans pass.
- [ ] Build artifact contains correct handler and dependencies.
- [ ] Environment variables and secrets are configured.
- [ ] IAM permissions are reviewed.
- [ ] Published version and alias strategy are defined.
- [ ] Smoke test succeeds after deployment.
- [ ] Alarms and rollback procedure are ready.

## Useful SAM commands

```bash
sam validate
sam build
sam local invoke MyFunction --event events/event.json
sam deploy --guided
sam logs -n MyFunction --tail
```

## Quick interview points

- Lambda timeout maximum: 15 minutes.
- Execution role: permissions for code to call AWS APIs.
- Resource policy: controls who can invoke the function.
- Cold start: environment initialization before handler execution.
- Reserved concurrency: reserves and caps a function's concurrency.
- Provisioned concurrency: pre-initializes execution environments.
- Alias: named pointer to a published version.
- Idempotency: repeated event processing does not create repeated side effects.
- DLQ/failure destination: captures failed events according to the invocation model.
- VPC attachment: enables access to VPC resources; internet access requires appropriate routing.

## Final production reminder

Lambda behavior differs by invocation type and event source. Always validate retry semantics, batching, partial failure support, timeout interactions, quotas, and pricing for the specific integration you are deploying.
