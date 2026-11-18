# AWS Lambda Insights

Lambda Insights is a monitoring and troubleshooting feature that provides system-level metrics and diagnostic information for Lambda execution environments.

## What can it help monitor?

Depending on runtime and configuration, Lambda Insights can provide information such as:

- CPU utilization
- Memory utilization
- Network activity
- Disk usage
- Execution environment performance
- Cold-start and runtime-related diagnostics

Use standard Lambda metrics for invocations, errors, duration, and throttles; use Lambda Insights for additional execution-environment visibility.

## Enable Lambda Insights

The general setup includes:

1. Open the Lambda function in the AWS Console.
2. Open the Monitoring and operations tools section.
3. Enable Lambda Insights.
4. Ensure the execution role has the permissions required by the Lambda Insights extension.
5. Deploy or update the function and invoke it.
6. View the Lambda Insights metrics and logs in CloudWatch.

The exact console labels and setup steps can change. Follow the current AWS documentation for the supported runtime and extension version.

## IAM permissions

AWS provides a managed policy commonly named `CloudWatchLambdaInsightsExecutionRolePolicy` for the extension's required permissions. Review the policy before attaching it and follow your organization's least-privilege standards.

## When should you use it?

Use Lambda Insights when standard CloudWatch metrics do not provide enough detail to investigate memory pressure, CPU usage, or execution-environment behavior.

## Best practices

- Enable it for functions where deeper runtime visibility is useful.
- Check supported runtimes and extension versions.
- Include the additional telemetry cost in your monitoring plan.
- Review the execution role permissions.
- Use Insights together with logs, native metrics, and tracing rather than as a replacement.
