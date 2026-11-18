# AWS Lambda Interview Preparation

Interview-focused notes for AWS Lambda, serverless architecture, DevOps automation, production operations, and troubleshooting.

## Study material

1. [Top 50 Lambda Interview Questions](Top-50-Lambda-Interview-Questions.md)
2. [Lambda Scenario-Based Questions](Lambda-Scenario-Based-Questions.md)
3. [Lambda Production Use Cases](Lambda-Production-Use-Cases.md)
4. [Lambda DevOps Cheat Sheet](Lambda-DevOps-Cheat-Sheet.md)

## Suggested preparation plan

| Session | Focus |
|---|---|
| 1 | Lambda fundamentals, execution model, triggers, concurrency |
| 2 | IAM, networking, security, observability |
| 3 | Production scenarios, retries, DLQs, idempotency |
| 4 | Deployment, CI/CD, versions, aliases, rollback |
| 5 | Explain two projects end-to-end using architecture diagrams |

## How to answer interview questions

Use this structure for technical and scenario questions:

1. **Context:** Explain the workload or problem.
2. **Design:** Describe the AWS services and event flow.
3. **Implementation:** Explain configuration, code, IAM, and deployment.
4. **Reliability/security:** Cover retries, idempotency, least privilege, monitoring, and failure handling.
5. **Outcome:** State measurable results only when you have real evidence.

## Important interview reminder

Do not memorize only definitions. Be ready to explain what happens when a Lambda function times out, is throttled, receives duplicate events, cannot reach a VPC resource, or deploys a faulty version. Verify current runtime support, service quotas, and pricing in AWS documentation before production decisions.
