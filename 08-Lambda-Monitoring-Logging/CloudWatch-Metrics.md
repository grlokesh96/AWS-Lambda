# AWS Lambda CloudWatch Metrics

Lambda automatically publishes service metrics to Amazon CloudWatch. These metrics help identify errors, latency, throttling, and concurrency pressure.

## Important metrics

| Metric | What it tells you |
|---|---|
| `Invocations` | Number of times the function was invoked |
| `Errors` | Number of invocations that returned a function error |
| `Duration` | Execution duration in milliseconds |
| `Throttles` | Number of invocation requests throttled |
| `ConcurrentExecutions` | Number of concurrent executions |
| `UnreservedConcurrentExecutions` | Account-level unreserved concurrency usage |
| `AsyncEventAge` | Age of asynchronous events waiting to be processed |
| `AsyncEventsDropped` | Asynchronous events discarded after retry/age handling |

Metrics are published in the `AWS/Lambda` namespace. Some metrics are available only for particular invocation types or event sources.

## View metrics using AWS CLI

```bash
aws cloudwatch list-metrics \
  --namespace AWS/Lambda \
  --dimensions Name=FunctionName,Value=demo-lambda
```

Get recent invocation statistics:

```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=demo-lambda \
  --start-time 2026-11-18T00:00:00Z \
  --end-time 2026-11-18T01:00:00Z \
  --period 300 \
  --statistics Sum
```

Replace the time range with the period you want to investigate.

## Calculate error rate

A useful basic measure is:

\[
\text{Error Rate (\%)} =
\frac{\text{Errors}}{\text{Invocations}} \times 100
\]

For example, if a function receives 10,000 invocations and 50 fail:

\[
\frac{50}{10000} \times 100 = 0.5\%
\]

For dashboards, make sure the error and invocation periods and dimensions match.

## Duration percentiles

Average duration can hide slow requests. Monitor percentile statistics such as p90, p95, or p99 where supported. Compare them with the function timeout and your application latency objective.

## Create a dashboard

A Lambda dashboard commonly includes:

- Invocations
- Errors and error rate
- Duration p50/p95/p99
- Throttles
- Concurrent executions
- Async event age, if applicable

## Best practices

- Monitor trends and percentiles, not just averages.
- Use dimensions consistently, such as function name and resource.
- Compare metrics with deployment timestamps.
- Set alarms against workload-specific service-level objectives.
- Avoid creating too many high-cardinality custom metrics.
