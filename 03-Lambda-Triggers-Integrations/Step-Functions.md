# AWS Step Functions with Lambda

## 1. Introduction

AWS Step Functions is a serverless workflow service.

It coordinates multiple tasks and AWS services into a workflow. Lambda functions can be used as tasks inside that workflow.

**Simple example:** One Lambda function validates an order, another processes payment, and another sends a notification.

## 2. Architecture

```text
        Application
             |
             v
       Step Functions
             |
             v
       Validate Order
        (Lambda 1)
             |
             v
       Process Payment
        (Lambda 2)
             |
             v
       Send Notification
        (Lambda 3)
             |
             v
        Workflow complete
```

## 3. How it works

1. A workflow execution starts.
2. Step Functions runs the first task.
3. The task invokes a Lambda function.
4. The function returns a result.
5. Step Functions moves to the next state.
6. The workflow continues until it succeeds or fails.

## 4. Common use cases

- Multi-step application workflows.
- Order processing.
- Data processing pipelines.
- Infrastructure automation.
- Approval workflows.
- Coordinating retries and error handling.

## 5. Important Step Functions concepts

| Term | Explanation |
|---|---|
| State machine | Defines the workflow |
| State | One step in the workflow |
| Task state | Performs work, such as invoking Lambda |
| Choice state | Selects a path based on input |
| Wait state | Pauses the workflow |
| Parallel state | Runs branches in parallel |
| Execution | One run of a state machine |

## 6. Simple workflow definition

The following Amazon States Language example invokes one Lambda function.

Replace the function ARN with your actual Lambda ARN.

```json
{
  "Comment": "Simple Lambda workflow",
  "StartAt": "ProcessTask",
  "States": {
    "ProcessTask": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "arn:aws:lambda:REGION:ACCOUNT_ID:function:lambda-demo",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "End": true
    }
  }
}
```

## 7. Create a state machine

Using the AWS Console:

1. Open AWS Step Functions.
2. Choose **Create state machine**.
3. Select the workflow type.
4. Define the workflow using the visual editor or Amazon States Language.
5. Configure the required IAM execution role.
6. Create the state machine.
7. Start a new execution.
8. Review the execution history and output.

## 8. Error handling

Step Functions supports workflow error handling.

Common fields include:

- `Retry`: Retry a task for selected errors.
- `Catch`: Handle an error and move to another state.
- `TimeoutSeconds`: Set a task timeout.
- `Next`: Move to another state.
- `End`: Finish the workflow.

Example structure:

```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::lambda:invoke",
  "Parameters": {
    "FunctionName": "YOUR_LAMBDA_ARN",
    "Payload.$": "$"
  },
  "Retry": [
    {
      "ErrorEquals": ["Lambda.ServiceException"],
      "IntervalSeconds": 2,
      "MaxAttempts": 3,
      "BackoffRate": 2
    }
  ],
  "End": true
}
```

Choose retryable errors carefully. Retrying a non-idempotent operation can create duplicate side effects.

## 9. Standard vs Express workflows

| Standard | Express |
|---|---|
| Designed for long-running workflows | Designed for high-volume, short-duration workflows |
| Execution history is retained longer | Execution history and monitoring differ |
| Supports long-running processes | Suitable for event processing and high-throughput workloads |

Check the current AWS documentation for exact duration, execution, and pricing differences.

## 10. Step Functions vs Lambda

| AWS Step Functions | AWS Lambda |
|---|---|
| Coordinates workflow steps | Executes application code |
| Manages workflow state and transitions | Processes an invocation |
| Supports retry and catch logic | Handles code-level errors |
| Can call multiple services | Can integrate with AWS services |

They work together rather than replacing each other.

## 11. DevOps use case

A deployment automation workflow can:

1. Validate deployment input.
2. Run a pre-deployment check.
3. Invoke a deployment function.
4. Verify application health.
5. Send a success or failure notification.

## 12. Interview questions

**Q1. What is AWS Step Functions?**

It is a serverless workflow service used to coordinate tasks and AWS services.

**Q2. Can Step Functions invoke Lambda?**

Yes. Lambda functions can be invoked as task states in a state machine.

**Q3. What is a state machine?**

It is the workflow definition that describes the states and transitions.

**Q4. What is the purpose of Retry and Catch?**

Retry attempts a task again for selected errors. Catch handles an error and routes the workflow to another state.

## Summary

Step Functions coordinates multiple tasks, while Lambda performs individual units of code execution. Together they help build reliable multi-step automation.
