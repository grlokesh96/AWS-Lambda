# AWS Lambda Reserved Concurrency

Reserved Concurrency reserves a portion of an account's available concurrency for a function and sets the maximum number of concurrent executions that function can use.

## Why use it?

- Protect a critical function from being starved by other functions.
- Limit traffic to protect a downstream database or API.
- Control the maximum scale of a workload.
- Isolate concurrency capacity between functions.

## Configure reserved concurrency

Set a reserved concurrency limit of 20:

```bash
aws lambda put-function-concurrency \
  --function-name demo-lambda \
  --reserved-concurrent-executions 20
```

View the configuration:

```bash
aws lambda get-function-concurrency \
  --function-name demo-lambda
```

Remove the reserved concurrency setting:

```bash
aws lambda delete-function-concurrency \
  --function-name demo-lambda
```

Removing the setting does not reserve capacity for the function; it returns to the unreserved pool.

## Reserved vs provisioned concurrency

| Feature | Reserved Concurrency | Provisioned Concurrency |
|---|---|---|
| Main purpose | Reserve and cap concurrency | Keep environments initialized |
| Limits maximum function concurrency | Yes | Not by itself |
| Reduces cold-start initialization | No | Yes, for provisioned capacity |
| Additional provisioned-capacity charge | No separate provisioned charge | Yes |
| Applies to | Function | Published version or alias |

The two features can be used together for different goals, subject to Lambda configuration constraints.

## Example scenario

A function writes to a database that safely supports 30 concurrent connections. Setting reserved concurrency to a suitable value can limit Lambda's concurrent execution pressure on that database. The exact limit should account for other clients and connection reuse.

## Best practices

- Set limits based on downstream capacity and account quotas.
- Monitor `ConcurrentExecutions` and `Throttles`.
- Remember that reserved concurrency set to zero intentionally disables function concurrency.
- Consider event source batching and retry behavior when throttling occurs.
- Do not assume reserved concurrency eliminates cold starts.
