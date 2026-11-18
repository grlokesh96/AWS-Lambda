# AWS Lambda Memory Exceeded Errors

A memory-related failure occurs when a Lambda execution environment needs more memory than the function has configured or when the runtime reports an out-of-memory condition.

## Common symptoms

- Runtime exits unexpectedly.
- Logs or reports indicate memory exhaustion.
- The function fails on large payloads or high-load requests.
- Memory usage approaches the configured limit.

## Step 1: Check configured memory

```bash
aws lambda get-function-configuration \
  --function-name demo-lambda \
  --query '{MemorySize:MemorySize,Timeout:Timeout}'
```

## Step 2: Review REPORT log lines

CloudWatch Logs may include a report similar to:

```text
REPORT RequestId: ... Duration: 1200 ms
Billed Duration: 1200 ms Memory Size: 512 MB
Max Memory Used: 480 MB
```

Compare `Max Memory Used` with `Memory Size`. This example indicates the function used most of its configured memory.

## Step 3: Increase memory for testing

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --memory-size 1024
```

Benchmark more than one setting. Increasing memory also increases the CPU resources available to the function.

## Step 4: Optimize application memory usage

- Process large files in chunks or streams.
- Avoid loading unnecessary data into memory.
- Release references to large temporary objects when they are no longer needed.
- Avoid retaining unbounded data in global variables.
- Review library and runtime memory overhead.
- Reduce payload size where possible.

## Step 5: Load test

Use representative payloads and concurrency. Record maximum memory use, duration, errors, and cost for each memory setting.

## Best practices

- Leave headroom above typical maximum memory use.
- Test worst-case payload sizes.
- Monitor memory and duration after deployments.
- Use Lambda Insights for additional execution-environment visibility where appropriate.
- Do not assume the smallest memory setting is the cheapest overall; faster execution can sometimes reduce total cost.
