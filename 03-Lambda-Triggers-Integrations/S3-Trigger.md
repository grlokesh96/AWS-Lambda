# AWS Lambda with S3 Trigger

## 1. Introduction

Amazon S3 is an object storage service. It can send event notifications when actions happen in a bucket.

A Lambda function can be invoked when an object is created, removed, or another supported S3 event occurs.

**Example:** When a user uploads an image, Lambda can automatically process it.

## 2. Architecture

```text
User uploads file
       |
       v
   S3 Bucket
       |
       | Object-created event
       v
 Lambda Function
       |
       | Process file
       v
 Output bucket / destination
       |
       v
 CloudWatch Logs
```

## 3. How it works

1. A user uploads a file to an S3 bucket.
2. S3 detects the configured event.
3. S3 sends an event notification.
4. Lambda is invoked with information about the object.
5. The function processes the file.
6. The function can save the result to another location.

## 4. Common use cases

- Image resizing.
- File format conversion.
- Log file processing.
- Data validation.
- Automatic document processing.
- Triggering downstream workflows.

## 5. Example Python Lambda function

This example prints the bucket and object key from an S3 event.

```python
import urllib.parse

def lambda_handler(event, context):
    for record in event.get("Records", []):
        bucket = record["s3"]["bucket"]["name"]
        key = urllib.parse.unquote_plus(
            record["s3"]["object"]["key"]
        )

        print(f"Bucket: {bucket}")
        print(f"Object: {key}")

    return {"statusCode": 200}
```

## 6. Configure an S3 trigger

Using the AWS Console:

1. Open S3.
2. Create or select a bucket.
3. Open the bucket's **Properties**.
4. Find **Event notifications**.
5. Create an event notification.
6. Select an event type, such as object creation.
7. Choose Lambda as the destination.
8. Select the Lambda function and save.

AWS must have permission to invoke the Lambda function. The Console can add the required resource-based permission when configuring the trigger.

## 7. Required IAM permissions

The Lambda execution role needs `s3:GetObject` if the function reads the uploaded file.

Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

Replace the bucket name. Add only the permissions the function actually needs.

## 8. Important warning: Recursive triggers

Be careful when the function writes output to the same bucket and the same event prefix that triggers it.

For example:

```text
S3 upload
   |
   v
Lambda runs
   |
   v
Lambda writes another file to same trigger path
   |
   v
S3 invokes Lambda again
```

This can create an invocation loop.

Use separate input and output buckets, or configure event filters such as prefixes and suffixes to prevent recursion.

## 9. Testing

Upload a file:

```bash
aws s3 cp sample.txt s3://YOUR-BUCKET-NAME/
```

Check Lambda logs in CloudWatch Logs:

```text
/aws/lambda/YOUR-FUNCTION-NAME
```

## 10. Interview questions

**Q1. How does S3 trigger Lambda?**

S3 sends an event notification to a configured Lambda destination when a matching bucket event occurs.

**Q2. What is an S3 event notification?**

It is a notification generated when a configured event happens in an S3 bucket.

**Q3. How do you avoid recursive Lambda invocations?**

Use separate input/output buckets or configure event filters so the function's output does not match its own trigger.

## Summary

S3 triggers allow Lambda to process files automatically when configured bucket events occur.
