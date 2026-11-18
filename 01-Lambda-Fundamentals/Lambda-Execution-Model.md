# AWS Lambda Execution Model

## 1. Introduction

The Lambda execution model explains what happens when AWS runs your function.

Lambda creates and manages an execution environment for your function. That environment provides the runtime and resources required to execute the code.

## 2. Main execution phases

A simplified execution has three phases:

### Phase 1: Initialization

Lambda prepares the execution environment.

During this phase, it can:

- Start the runtime.
- Load your function code.
- Load dependencies.
- Run initialization code outside the handler.

### Phase 2: Invocation

Lambda calls your handler with the event and context.

The handler performs the task and returns a result or raises an error.

### Phase 3: Shutdown

Lambda may shut down the execution environment when it is no longer needed. The timing is managed by AWS and should not be relied on for application cleanup.

## 3. Execution flow

```text
Event arrives
     |
     v
Environment ready?
     |
  +--+--+
  |     |
 Yes    No
  |     |
  |   Initialize
  |     |
  +--+--+
     |
     v
Invoke handler
     |
     v
Execute function
     |
     v
Return result / error
     |
     v
Environment may be reused
or shut down later
```

## 4. Cold start

A cold start happens when Lambda needs to prepare a new execution environment before running the handler.

This preparation can add latency.

Common reasons include:

- A new environment is required.
- The function has large dependencies.
- Initialization code takes a long time.
- The function uses a large deployment package.

## 5. Warm execution

A warm execution happens when Lambda reuses an existing execution environment.

The environment may already have the runtime and dependencies loaded, so it can often start the handler faster.

**Important:** AWS may reuse an environment, but reuse is not guaranteed.

## 6. Simple Python example

```python
import json

# Initialization code
print("Lambda environment initialized")

def lambda_handler(event, context):
    print("Function invoked")

    return {
        "statusCode": 200,
        "body": json.dumps("Hello from AWS Lambda")
    }
```

- Code outside the handler runs during initialization.
- `lambda_handler` is called for each invocation.
- `event` contains the input data.
- `context` provides information about the current invocation.

## 7. Best practices

- Keep initialization code lightweight.
- Reuse SDK clients and database connections where appropriate.
- Do not store important user data in local memory between invocations.
- Make functions safe to retry.
- Avoid depending on the environment remaining active.
- Use connection timeouts and handle failures properly.

## 8. Interview questions

**Q1. What is a cold start?**

A cold start occurs when Lambda prepares a new execution environment before invoking the function.

**Q2. What is a warm execution?**

It is an invocation that uses an existing execution environment that Lambda has retained and reused.

**Q3. Is a warm execution guaranteed?**

No. AWS controls execution environment reuse.

**Q4. What is the Lambda handler?**

The handler is the function entry point that Lambda calls when the function is invoked.

## Summary

Understand initialization, invocation, and shutdown. This helps when troubleshooting function latency and designing reliable Lambda applications.
