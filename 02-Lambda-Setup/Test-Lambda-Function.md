# Test an AWS Lambda Function

## 1. Introduction

Testing confirms that your Lambda function runs correctly and returns the expected result.

You can test Lambda functions using:

- AWS Management Console.
- AWS CLI.
- Automated tests in a CI/CD pipeline.
- Integration tests with connected AWS services.

## 2. Prerequisites

- AWS CLI installed and configured.
- A Lambda function named `lambda-demo`.
- Permission to invoke the function.
- A test event in JSON format.

## 3. Create a test event

Create a file named `event.json`:

```json
{
  "name": "Lokesh"
}
```

## 4. Invoke the Lambda function

Run:

```bash
aws lambda invoke \
  --function-name lambda-demo \
  --payload fileb://event.json \
  response.json
```

The function response is saved in `response.json`.

## 5. Check the response

```bash
cat response.json
```

Expected response:

```json
{
  "statusCode": 200,
  "body": "Hello, Lokesh! Function deployed successfully."
}
```

Your exact response depends on the code currently deployed.

## 6. Test using the AWS Console

1. Open AWS Lambda in the AWS Console.
2. Select `lambda-demo`.
3. Open the **Test** tab or test section.
4. Create a new test event.
5. Add the JSON event.
6. Save the event.
7. Choose **Test**.
8. Review the execution result.

## 7. Check CloudWatch Logs

Lambda sends logs to CloudWatch Logs when the function has the required logging permissions.

The default log group name is:

```text
/aws/lambda/lambda-demo
```

Open the CloudWatch console:

1. Search for **CloudWatch**.
2. Select **Logs**.
3. Open **Log groups**.
4. Select `/aws/lambda/lambda-demo`.
5. Open the latest log stream.

You can also use AWS CLI:

```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/lambda-demo
```

## 8. Common Lambda test errors

| Error | Possible reason | What to check |
|---|---|---|
| ResourceNotFoundException | Function not found | Function name and region |
| AccessDeniedException | Caller cannot invoke function | IAM invoke permission |
| Task timed out | Function took too long | Timeout, code, and dependencies |
| Runtime.ImportModuleError | Dependency missing | ZIP package structure and runtime |
| AccessDenied in function logs | Execution role lacks permission | IAM policy and resource policy |
| Invalid JSON | Event payload is malformed | JSON syntax and input format |

## 9. Test with different inputs

Example event:

```json
{
  "name": "AWS DevOps Engineer"
}
```

Another example:

```json
{
  "name": "Cloud Engineer"
}
```

Try an empty event too:

```json
{}
```

The function should use its default name when the `name` field is missing.

## 10. Automated test example

You can test the Python handler locally with `pytest`.

Create `test_lambda_function.py`:

```python
from lambda_function import lambda_handler

def test_lambda_handler():
    result = lambda_handler(
        {"name": "DevOps Engineer"},
        None
    )

    assert result["statusCode"] == 200
    assert "DevOps Engineer" in result["body"]

def test_default_name():
    result = lambda_handler({}, None)

    assert result["statusCode"] == 200
    assert "DevOps Engineer" in result["body"]
```

Install and run pytest:

```bash
python3 -m pip install pytest
python3 -m pytest -v
```

These are local unit tests. They do not replace integration tests against AWS services.

## 11. Useful verification commands

```bash
# Check the function configuration
aws lambda get-function-configuration \
  --function-name lambda-demo

# Invoke the function
aws lambda invoke \
  --function-name lambda-demo \
  --payload '{}' \
  response.json

# View recent log streams
aws logs describe-log-streams \
  --log-group-name /aws/lambda/lambda-demo \
  --order-by LastEventTime \
  --descending \
  --max-items 5
```

## 12. DevOps testing checklist

- [ ] Function exists in the expected AWS region.
- [ ] Function code is deployed successfully.
- [ ] Test event is valid JSON.
- [ ] Function returns the expected output.
- [ ] CloudWatch Logs show the execution.
- [ ] No unexpected errors or timeouts occur.
- [ ] IAM permissions follow least privilege.
- [ ] Automated tests pass before deployment.

## 13. Interview questions

**Q1. How do you test a Lambda function?**

Using the AWS Console, AWS CLI, local unit tests, and integration tests.

**Q2. Where do you check Lambda execution logs?**

In the CloudWatch Logs group `/aws/lambda/<function-name>`, provided logging is configured and permitted.

**Q3. How do you troubleshoot a failed invocation?**

Check the invocation response, CloudWatch Logs, function metrics, timeout, runtime, event payload, and IAM permissions.

## Summary

Testing verifies that the deployed function works as expected. Always check the response, logs, errors, and permissions before considering a deployment complete.
