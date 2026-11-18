# Python Basics for AWS Lambda

AWS Lambda runs Python code in response to events. The handler is the entry point that Lambda calls for each invocation.

## Basic handler

```python
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Hello from AWS Lambda"
    }
```

- `event`: Input data from the trigger.
- `context`: Runtime information such as request ID and remaining execution time.

## Read event data

Example event:

```json
{
  "name": "Lokesh",
  "environment": "dev"
}
```

Handler:

```python
def lambda_handler(event, context):
    name = event.get("name", "Guest")
    environment = event.get("environment", "dev")

    return {
        "message": f"Hello {name}",
        "environment": environment,
        "request_id": context.aws_request_id
    }
```

## Environment variables

```python
import os

APP_ENV = os.environ.get("APP_ENV", "development")

def lambda_handler(event, context):
    return {"environment": APP_ENV}
```

Use environment variables for configuration. Store passwords and API tokens in Secrets Manager or Parameter Store rather than embedding them in code.

## Logging

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info("Request started: %s", context.aws_request_id)
    return {"status": "ok"}
```

Avoid logging credentials, tokens, or sensitive payload fields.

## Exception handling

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        value = int(event["value"])
        return {"result": value * 2}
    except (KeyError, ValueError):
        logger.exception("Invalid input")
        raise
```

Re-raising an exception allows Lambda and the configured invocation mechanism to recognize the failure and apply its retry behavior.

## Local test

```python
def lambda_handler(event, context):
    return {"message": "Hello", "input": event}
```

You can test with the AWS Console test event, AWS CLI `invoke`, or a local test framework. A local mock context is useful for unit tests.

## Best practices

- Keep handlers small and move business logic into reusable functions.
- Initialize reusable SDK clients outside the handler.
- Validate event structure.
- Use structured logs and request IDs.
- Write unit tests for normal, invalid, and failure cases.
