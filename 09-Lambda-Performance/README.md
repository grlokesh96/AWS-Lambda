# AWS Lambda Performance Optimization

This folder covers practical ways to improve Lambda startup time, execution duration, concurrency behavior, memory sizing, timeout settings, and failure handling.

## Contents

| File | What you will learn |
|---|---|
| [Cold-Start-Optimization.md](Cold-Start-Optimization.md) | Reduce initialization latency |
| [Provisioned-Concurrency.md](Provisioned-Concurrency.md) | Keep execution environments initialized |
| [Reserved-Concurrency.md](Reserved-Concurrency.md) | Reserve and limit function concurrency |
| [Memory-Optimization.md](Memory-Optimization.md) | Tune memory, CPU, and cost |
| [Timeout-Configuration.md](Timeout-Configuration.md) | Set safe execution timeouts |
| [Retry-Mechanism.md](Retry-Mechanism.md) | Understand retries for different invocation types |
| [Dead-Letter-Queue.md](Dead-Letter-Queue.md) | Capture failed asynchronous events |

## Performance tuning workflow

```text
Measure Baseline
      |
      v
Review Duration / Errors / Throttles
      |
      v
Identify Bottleneck
      |
      +---- Cold start?       -> Optimize initialization / Provisioned Concurrency
      |
      +---- Slow execution?   -> Tune memory / code / downstream calls
      |
      +---- Throttling?       -> Review concurrency and downstream capacity
      |
      +---- Failed events?    -> Configure retries / DLQ or destinations
      |
      v
Change One Variable at a Time
      |
      v
Load Test and Compare Cost
      |
      v
Monitor in Production
```

## Important concepts

- **Cold start:** Time needed to initialize an execution environment before handling an invocation.
- **Provisioned concurrency:** Pre-initialized execution environments for predictable startup latency.
- **Reserved concurrency:** A function-level concurrency allocation and upper limit.
- **Memory:** Lambda memory configuration also affects available CPU and other resources.
- **Timeout:** Maximum allowed duration for one invocation.
- **Retries:** Depend on whether invocation is synchronous, asynchronous, or event-source mapped.
- **Dead-letter queue:** A destination for failed asynchronous events after retry handling.

## Best practices

- Measure p95/p99 duration and cold-start impact before tuning.
- Load test using realistic payloads and concurrency.
- Set timeouts based on downstream service limits and application objectives.
- Make event processing idempotent because retries can cause duplicate processing.
- Configure alarms for errors, throttles, duration, and event age where applicable.
- Review cost as well as latency after every performance change.
