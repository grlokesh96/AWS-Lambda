# AWS Cost Optimization with Lambda

## 1. Introduction

AWS cost optimization means using cloud resources efficiently while meeting application performance, availability, and business requirements.

Lambda can automate cost reporting and approved operational actions. AWS Cost Explorer, AWS Budgets, Compute Optimizer, and native service features provide useful cost and usage information.

## 2. Architecture

```text
EventBridge Scheduler
        |
        v
   Lambda Function
        |
        v
   AWS cost APIs
        |
        v
 Analyze usage and cost
        |
        v
 Generate report
        |
        v
 SNS / Email notification
```

## 3. Common cost optimization opportunities

| Area | Opportunity |
|---|---|
| EC2 | Right-size instances and stop eligible non-production instances |
| EBS | Review unattached volumes and unused snapshots |
| S3 | Apply lifecycle rules to eligible objects |
| ECR | Apply image retention policies |
| Lambda | Optimize memory and execution duration |
| RDS | Review instance sizing and idle resources |
| Data transfer | Review unnecessary cross-region and internet transfer |

Do not assume a resource is wasteful just because it has low utilization. Consider workload patterns, redundancy, recovery, and business requirements.

## 4. Example: Read cost data using Boto3

This example requests daily unblended cost data for the previous calendar month.

```python
import boto3
from datetime import date, timedelta

ce = boto3.client("ce", region_name="us-east-1")

def lambda_handler(event, context):
    today = date.today()
    first_day_this_month = today.replace(day=1)
    last_day_previous_month = first_day_this_month - timedelta(days=1)
    first_day_previous_month = last_day_previous_month.replace(day=1)

    response = ce.get_cost_and_usage(
        TimePeriod={
            "Start": first_day_previous_month.isoformat(),
            "End": first_day_this_month.isoformat()
        },
        Granularity="DAILY",
        Metrics=["UnblendedCost"]
    )

    for result in response.get("ResultsByTime", []):
        print(result["TimePeriod"], result["Total"])

    return {"message": "Cost report generated"}
```

Cost Explorer API usage may incur charges. Check current AWS pricing and permissions before enabling regular reports. The Cost Explorer API is accessed through the billing endpoint, commonly in `us-east-1`.

## 5. Required permissions

For the example above, the Lambda execution role needs:

```text
ce:GetCostAndUsage
```

Billing data access may also depend on account-level billing permissions and settings.

## 6. Useful AWS CLI commands

Get cost and usage data:

```bash
aws ce get-cost-and-usage \
  --time-period Start=2026-10-01,End=2026-11-01 \
  --granularity MONTHLY \
  --metrics UnblendedCost
```

The end date is exclusive. Adjust dates to the reporting period you need.

## 7. Recommended automation workflow

1. Collect cost and usage data.
2. Group costs by service, account, or tag.
3. Compare current usage with previous periods.
4. Identify unusual increases.
5. Generate a report.
6. Notify resource owners.
7. Review recommendations.
8. Apply approved changes.
9. Monitor the result.

## 8. Best practices

- Use AWS Budgets for budget thresholds and alerts.
- Use Cost Explorer for cost analysis.
- Use Compute Optimizer for supported resource recommendations.
- Apply cost allocation tags.
- Review Savings Plans and Reserved Instances where appropriate.
- Automate non-production schedules with owner approval.
- Avoid automatically terminating resources based only on low usage.
- Track savings and application impact after changes.

## 9. Interview questions

**Q1. How can Lambda help reduce AWS costs?**

It can automate reports, identify cleanup candidates, and perform approved actions such as stopping eligible non-production instances.

**Q2. What is AWS Cost Explorer?**

It is a service for viewing and analyzing AWS costs and usage.

**Q3. What is the difference between cost monitoring and cost optimization?**

Monitoring identifies and tracks spending. Optimization changes resource usage or purchasing decisions to improve cost efficiency.

**Q4. Should cost automation delete resources automatically?**

Not without validated ownership, dependency checks, retention requirements, and approval.

## Summary

Lambda can automate cost reporting and approved optimization tasks. Combine it with Cost Explorer, AWS Budgets, tagging, and careful operational controls.
