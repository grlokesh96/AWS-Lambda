# AWS Lambda Monitoring and Logging

This folder explains how to monitor Lambda functions using CloudWatch Logs, metrics, alarms, AWS X-Ray, Lambda Insights, and SNS notifications.

## Contents

| File | What you will learn |
|---|---|
| [CloudWatch-Logs.md](CloudWatch-Logs.md) | View, query, and retain Lambda logs |
| [CloudWatch-Metrics.md](CloudWatch-Metrics.md) | Understand invocation, duration, error, and throttle metrics |
| [CloudWatch-Alarm.md](CloudWatch-Alarm.md) | Create alarms for Lambda health |
| [AWS-X-Ray.md](AWS-X-Ray.md) | Trace requests across distributed services |
| [Lambda-Insights.md](Lambda-Insights.md) | Monitor runtime and execution environment performance |
| [Error-Monitoring.md](Error-Monitoring.md) | Detect, investigate, and reduce Lambda errors |
| [SNS-Notifications.md](SNS-Notifications.md) | Send monitoring alerts through SNS |

## Monitoring architecture

```text
Users / Event Sources
        |
        v
  Lambda Function
    |      |       \
    |      |        \
    v      v         v
 CloudWatch  X-Ray  Lambda Insights
   Logs     Traces    Metrics
     \       |       /
      \      |      /
       CloudWatch Metrics
              |
          CloudWatch
            Alarm
              |
              v
             SNS
              |
       Email / On-call
```

## Key Lambda metrics

| Metric | Meaning |
|---|---|
| `Invocations` | Number of function invocations |
| `Errors` | Invocations that ended with a function error |
| `Duration` | Time spent executing the function |
| `Throttles` | Invocation requests rejected due to concurrency limits |
| `ConcurrentExecutions` | Number of concurrent executions |
| `DeadLetterErrors` | Errors while sending failed asynchronous events to a dead-letter destination |
| `IteratorAge` | Age of the last record in the event source stream, for supported stream mappings |

## Recommended practices

- Write structured logs with request IDs and useful context.
- Never log passwords, tokens, or sensitive personal data.
- Set log retention instead of keeping logs forever by default.
- Create alarms for errors, throttles, and latency based on workload requirements.
- Use X-Ray when distributed request tracing is useful.
- Use Lambda Insights when runtime and execution-environment metrics are needed.
- Test alert delivery and document an incident response process.

> Metric availability and dimensions depend on the event source and Lambda configuration. Select thresholds based on a measured baseline, not arbitrary values.
