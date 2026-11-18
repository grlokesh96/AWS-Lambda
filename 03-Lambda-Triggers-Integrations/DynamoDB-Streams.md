# AWS Lambda with DynamoDB Streams

## 1. Introduction

Amazon DynamoDB is a managed NoSQL database.

DynamoDB Streams captures changes made to items in a DynamoDB table. Lambda can read these stream records and run code when table changes occur.

## 2. Architecture

```text
Application
     |
     | Insert / Update / Delete
     v
DynamoDB Table
     |
     | Change record
     v
DynamoDB Stream
     |
     | Lambda polls stream
     v
Lambda Function
     |
     v
Process change
```

## 3. How it works

1. An application inserts, updates, or deletes an item.
2. DynamoDB records the change in the stream.
3. Lambda polls the stream using an event source mapping.
4. Lambda receives a batch of records.
5. The function processes the changes.

## 4. Common use cases

- React to database changes.
- Send notifications when an order is created.
- Synchronize data with another service.
- Maintain search indexes.
- Trigger downstream processing.
- Build event-driven applications.

## 5. Enable DynamoDB Streams

Using the AWS Console:

1. Open DynamoDB.
2. Select your table.
3. Open the **Exports and streams** or stream settings section.
4. Enable DynamoDB Streams.
5. Select the required stream view type.
6. Save the configuration.

Common stream view types:

| View type | Information captured |
|---|---|
| KEYS_ONLY | Only the item's key attributes |
| NEW_IMAGE | The item after the change |
| OLD_IMAGE | The item before the change |
| NEW_AND_OLD_IMAGES | Both old and new item images |

## 6. Connect Lambda to the stream

1. Open the Lambda function.
2. Choose **Add trigger**.
3. Select DynamoDB.
4. Choose the table stream ARN.
5. Configure batch size and starting position.
6. Add the trigger.

The Lambda execution role needs the required DynamoDB Streams read permissions.

## 7. Example Python Lambda function

```python
def lambda_handler(event, context):
    for record in event.get("Records", []):
        event_name = record.get("eventName")
        print("DynamoDB event:", event_name)

        if event_name == "INSERT":
            print("A new item was inserted")

        elif event_name == "MODIFY":
            print("An item was updated")

        elif event_name == "REMOVE":
            print("An item was deleted")

    return {"statusCode": 200}
```

## 8. Important concepts

- DynamoDB Streams retains stream records for a limited period.
- Lambda polls the stream through an event source mapping.
- Records are processed in batches.
- Ordering is maintained for the same item key.
- Failed processing can cause retries.
- Duplicate processing is possible, so design handlers to be idempotent.

## 9. DynamoDB Streams vs SQS

| DynamoDB Streams | SQS |
|---|---|
| Captures changes to DynamoDB items | Stores application messages |
| Used for database change processing | Used for queue-based processing |
| Records have a limited retention period | Queue retention is configurable |
| Lambda reads stream records | Lambda polls queue messages |

## 10. Interview questions

**Q1. What is DynamoDB Streams?**

It captures changes made to items in a DynamoDB table.

**Q2. How does Lambda connect to DynamoDB Streams?**

Lambda uses an event source mapping to poll the stream and process records.

**Q3. What are the stream view types?**

Keys only, new image, old image, and new and old images.

**Q4. Can duplicate records be processed?**

Yes. Applications should be designed to handle retries and duplicate processing safely.

## Summary

DynamoDB Streams and Lambda allow applications to react automatically to database changes.
