# AWS Lambda with API Gateway

## 1. Introduction

Amazon API Gateway is an AWS service that allows applications to create and manage APIs.

API Gateway can receive an HTTP request and invoke a Lambda function to process it.

**Simple example:** A user calls an API URL. API Gateway sends the request to Lambda. Lambda processes the request and returns a response.

## 2. Architecture

```text
Client / Browser
       |
       | HTTP request
       v
  API Gateway
       |
       | Invoke Lambda
       v
  Lambda Function
       |
       | Response
       v
  API Gateway
       |
       v
Client receives response
```

## 3. How it works

1. A client sends an HTTP request.
2. API Gateway receives the request.
3. API Gateway invokes the configured Lambda function.
4. Lambda processes the request.
5. Lambda returns a response.
6. API Gateway sends the response to the client.

## 4. Common use cases

- Build REST APIs.
- Create backend services for web applications.
- Build serverless applications.
- Process API requests without managing servers.
- Create lightweight automation endpoints.

## 5. Create an HTTP API using the AWS Console

1. Open the AWS Management Console.
2. Search for **API Gateway**.
3. Choose **Create API**.
4. Select **HTTP API**.
5. Add a Lambda integration.
6. Select your Lambda function.
7. Configure a route, for example `GET /hello`.
8. Create and deploy the API.
9. Copy the API endpoint.
10. Test it using a browser or `curl`.

The Console workflow can change. Follow the current AWS API Gateway guide if a screen looks different.

## 6. Example Lambda function

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json"
        },
        "body": json.dumps({
            "message": "Hello from Lambda",
            "method": event.get("requestContext", {})
                           .get("http", {})
                           .get("method")
        })
    }
```

The event structure depends on the API Gateway integration and payload format version.

## 7. Test the API

After deployment, use the API endpoint provided by API Gateway:

```bash
curl https://YOUR_API_ID.execute-api.YOUR_REGION.amazonaws.com/hello
```

Replace the placeholders with your actual API ID and region.

## 8. Permissions

API Gateway needs permission to invoke the Lambda function. When you create an integration through the Console, AWS can configure this permission for you.

Check the Lambda resource policy if you receive an invocation permission error.

## 9. Important points

- API Gateway handles HTTP requests and routes.
- Lambda executes the backend code.
- Configure authentication and authorization where required.
- Validate request input.
- Avoid returning sensitive information in API responses.
- Monitor API Gateway and Lambda errors.

## 10. Interview questions

**Q1. What is API Gateway?**

It is an AWS service used to create, publish, secure, and manage APIs.

**Q2. How does API Gateway integrate with Lambda?**

API Gateway invokes a Lambda function when a configured API route receives a request.

**Q3. What is the difference between API Gateway and Lambda?**

API Gateway manages API requests. Lambda runs the backend code.

## Summary

API Gateway and Lambda are commonly used together to build serverless HTTP APIs.
