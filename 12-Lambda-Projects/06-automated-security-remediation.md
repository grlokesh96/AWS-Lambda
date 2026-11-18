# Project 06: Automated Security Remediation

## Objective

Detect selected AWS security events and run safe, predefined remediation actions. Examples include notifying owners, quarantining a non-production security group after approval, or enforcing required S3 public-access settings.

Start with alert-only mode. Automated changes can interrupt production workloads if findings are misclassified.

## Architecture

```text
AWS Config / Security Hub / CloudTrail
                 |
                 v
             EventBridge
                 |
                 v
          Lambda triage
                 |
       +---------+----------+
       |                    |
       v                    v
  SNS notification     Approved rule?
                            |
                            v
                    Remediation action
                            |
                            v
                   Audit log / alert
```

## Example: report an S3 public-access finding

This sample receives an event and sends a notification. It deliberately does not modify the bucket.

```python
import json
import os
import boto3

sns = boto3.client("sns")
TOPIC_ARN = os.environ["SNS_TOPIC_ARN"]

def lambda_handler(event, context):
    detail = event.get("detail", {})
    message = {
        "source": event.get("source"),
        "detail_type": event.get("detail-type"),
        "account": event.get("account"),
        "region": event.get("region"),
        "finding": detail
    }

    sns.publish(
        TopicArn=TOPIC_ARN,
        Subject="AWS security event requires review",
        Message=json.dumps(message, default=str)
    )

    return {"status": "notification_sent"}
```

Configure an EventBridge rule to match the specific event source and finding type you intend to process. Event schemas differ between AWS Config, Security Hub, and CloudTrail; inspect a real sample event before writing filters.

## Remediation pattern

A safe remediation workflow should:

1. Validate the event source, account, region, resource ID, and finding type.
2. Check whether the resource is in an approved scope.
3. Deduplicate repeated events.
4. Determine whether remediation is reversible.
5. Notify the owner or request approval.
6. Execute only a fixed, allow-listed action.
7. Verify the post-change state.
8. Record the action, actor, reason, and outcome.

## Example remediation candidates

| Finding | Possible action | Important guardrail |
|---|---|---|
| S3 public access enabled | Apply account/bucket block-public-access settings | Confirm intended public website/data use |
| Security group open to `0.0.0.0/0` on admin port | Alert and request rule removal | Check approved bastion/VPN access |
| Unencrypted EBS volume | Plan migration to encrypted volume | Existing volume encryption cannot simply be toggled in place |
| IAM access key too old | Notify owner and rotate through a controlled workflow | Avoid disabling active production credentials blindly |
| CloudTrail disabled | Re-enable through a governed process | Validate organization-level trail configuration |

## IAM and security

Use separate roles for detection and remediation. The triage role should usually be read-only plus notification access. The remediation role should contain only the exact API actions required for a specific control, scoped to approved resources where possible.

Do not build a generic Lambda that accepts arbitrary AWS API names or arbitrary resource IDs from an event. Treat event payloads as untrusted input.

## Testing

- Replay representative events in a sandbox.
- Test malformed and duplicate events.
- Verify alert delivery and audit logs.
- Run remediation in dry-run mode first.
- Test rollback or recovery procedures.
- Confirm the function cannot affect resources outside its allow-list.

## Production recommendations

Use AWS Security Hub and AWS Config managed capabilities where appropriate. Consider Systems Manager Automation for standardized remediation runbooks. Keep human approval for high-impact changes and use CloudTrail to audit API activity.
