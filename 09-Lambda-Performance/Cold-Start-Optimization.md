# AWS Lambda Cold-Start Optimization

A cold start occurs when Lambda creates a new execution environment and initializes the runtime and function code before processing an invocation. The initialization work can add latency to the first request handled by that environment.

## Cold-start flow

```text
Invocation Arrives
       |
       v
Create / Prepare Execution Environment
       |
       v
Start Runtime
       |
       v
Load Code and Dependencies
       |
       v
Run Initialization Code
       |
       v
Invoke Handler
```

A warm invocation can reuse an existing execution environment, but reuse is not guaranteed.

## Common causes

- Large deployment packages or container images.
- Many dependencies imported during initialization.
- Expensive initialization code.
- VPC networking setup and other external dependency initialization.
- Runtime and architecture characteristics.
- Sudden traffic spikes that require new execution environments.

## Optimization techniques

### 1. Reduce package size

Remove unused libraries and avoid bundling unnecessary files.

### 2. Initialize reusable clients outside the handler

```python
import boto3

s3 = boto3.client("s3")

def lambda_handler(event, context):
    # Reuse the client when the execution environment is reused.
    return {"statusCode": 200}
```

Do not store per-request sensitive data in global variables.

### 3. Avoid unnecessary initialization

Load only the modules and configuration needed by the function. Move expensive work out of initialization when it can safely happen later.

### 4. Use an appropriate runtime and architecture

Benchmark supported runtimes and architectures with your actual workload. Do not choose based on assumptions alone.

### 5. Consider Provisioned Concurrency

For latency-sensitive functions with predictable traffic, provisioned concurrency can keep execution environments initialized. See [Provisioned-Concurrency.md](Provisioned-Concurrency.md).

## Measure initialization duration

Lambda logs may include a `REPORT` line with initialization-related details for cold starts. Review CloudWatch logs and metrics, and use tracing or Lambda Insights when needed.

## Best practices

- Measure cold-start frequency and user-visible impact.
- Keep deployment packages lean.
- Reuse SDK clients and database connections where safe.
- Avoid slow network calls during module initialization.
- Use Provisioned Concurrency only when its latency benefit justifies the cost.
- Load test burst traffic and scale-out behavior.
