# CloudWatch Automation with Python and Boto3

Boto3 allows Lambda to retrieve CloudWatch metrics and create or update monitoring resources. This example reads Lambda invocation statistics.

## IAM permissions

For metric retrieval, the execution role needs:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["cloudwatch:GetMetricStatistics"],
    "Resource": "*"
  }]
}
```

CloudWatch metric read APIs commonly require `"Resource": "*"`. Add other actions only if the function needs them.

## Read Lambda invocation metrics

```python
import boto3
from datetime import datetime, timedelta, timezone

cloudwatch = boto3.client("cloudwatch", region_name="ap-south-1")

def lambda_handler(event, context):
    function_name = event["function_name"]
    end_time = datetime.now(timezone.utc)
    start_time = end_time - timedelta(hours=1)

    response = cloudwatch.get_metric_statistics(
        Namespace="AWS/Lambda",
        MetricName="Invocations",
        Dimensions=[
            {"Name": "FunctionName", "Value": function_name}
        ],
        StartTime=start_time,
        EndTime=end_time,
        Period=300,
        Statistics=["Sum"]
    )

    datapoints = sorted(
        response.get("Datapoints", []),
        key=lambda item: item["Timestamp"]
    )

    return {
        "function_name": function_name,
        "datapoints": [
            {
                "timestamp": point["Timestamp"].isoformat(),
                "invocations": point["Sum"]
            }
            for point in datapoints
        ]
    }
```

## Read errors instead

Change:

```python
MetricName="Invocations"
```

to:

```python
MetricName="Errors"
```

Other common Lambda metrics include `Duration`, `Throttles`, and `ConcurrentExecutions`. Use the correct statistic and dimensions for each metric.

## Create an alarm

A Lambda function can also create a CloudWatch alarm using `cloudwatch.put_metric_alarm`. This requires `cloudwatch:PutMetricAlarm` and should be restricted through governance and approval controls because it changes monitoring configuration.

## Best practices

- Use UTC timestamps for metric queries.
- Choose a period appropriate to the metric and investigation window.
- Sort returned datapoints by timestamp; API results are not guaranteed to be chronological.
- Avoid hardcoding function names when an event or configuration can provide them safely.
- Use least-privilege permissions.
- Do not create alarms dynamically without controls that prevent duplicate or noisy alarms.
