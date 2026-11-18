# Project 05: AWS Cost Optimization Agent

## Objective

Build a serverless cost optimization assistant that collects cost and utilization signals, identifies potential savings, and produces recommendations for human review.

The first version should be **read-only**. Do not let an AI model directly terminate, resize, or delete infrastructure.

## Architecture

```text
EventBridge schedule
        |
        v
 Lambda cost collector
        |
        +----> Cost Explorer API
        |
        +----> CloudWatch metrics (optional)
        |
        v
Rule-based recommendation engine
        |
        +----> S3 report / DynamoDB
        |
        v
SNS email notification
        |
        v
Human approval before any change
```

Optional extension: send summarized findings to Amazon Bedrock for natural-language explanations. Keep deterministic calculations and policy checks outside the model.

## Example: monthly service cost report

Enable Cost Explorer in the AWS account before using its API. Cost data may have a reporting delay.

```python
import boto3
from datetime import date, timedelta

ce = boto3.client("ce")

def lambda_handler(event, context):
    end = date.today()
    start = end.replace(day=1)

    response = ce.get_cost_and_usage(
        TimePeriod={
            "Start": start.isoformat(),
            "End": end.isoformat()
        },
        Granularity="MONTHLY",
        Metrics=["UnblendedCost"],
        GroupBy=[{"Type": "DIMENSION", "Key": "SERVICE"}]
    )

    report = []
    for period in response.get("ResultsByTime", []):
        for group in period.get("Groups", []):
            amount = float(group["Metrics"]["UnblendedCost"]["Amount"])
            report.append({
                "service": group["Keys"][0],
                "cost": round(amount, 2),
                "unit": group["Metrics"]["UnblendedCost"]["Unit"]
            })

    report.sort(key=lambda item: item["cost"], reverse=True)
    return {"period_start": start.isoformat(), "service_costs": report}
```

For a complete monthly report, account for pagination, billing time zones, linked accounts, credits, refunds, and the metric appropriate to your finance process. Cost Explorer's `End` date is exclusive.

## Recommendation examples

| Signal | Recommendation | Safety check |
|---|---|---|
| EC2 low CPU over a sustained period | Review instance rightsizing | Check memory, network, business schedule |
| EBS unattached volume | Confirm ownership and backup, then review deletion | Never delete solely from one signal |
| Old ECR images | Apply lifecycle policy | Preserve deployed and rollback images |
| Idle load balancer | Review target health and traffic | Confirm no failover or maintenance use |
| Oversized non-prod schedule | Stop outside working hours | Confirm owner and maintenance window |

## Agent workflow

1. Collect billing and utilization data.
2. Normalize findings with resource ARN, account, region, tags, and evidence.
3. Apply deterministic rules and confidence thresholds.
4. Generate a report with estimated savings and assumptions.
5. Notify resource owners.
6. Require approval through an established change process.
7. Execute approved actions using a separate, tightly scoped role.
8. Record before/after state and verify the result.

## IAM permissions

For the report-only function, grant only required Cost Explorer read permissions (for example, `ce:GetCostAndUsage`) and read-only CloudWatch/resource discovery permissions if used. SNS publishing and S3 report writes should be limited to the specific resources.

## Security and governance

- Do not send secrets, customer data, or unrestricted account metadata to an LLM.
- Require evidence and explain uncertainty in every recommendation.
- Use allow-lists, budgets, approval gates, and change windows.
- Keep recommendation generation separate from remediation execution.
- Measure realized savings against a baseline; do not present estimated savings as guaranteed savings.

## Extension ideas

- Store recommendations in DynamoDB with `PENDING`, `APPROVED`, and `REJECTED` states.
- Add an approval API using API Gateway and Cognito/IAM authorization.
- Use Step Functions for multi-step approval and execution.
- Use Bedrock only to summarize findings, not to bypass policy controls.
