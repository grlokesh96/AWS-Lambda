# AWS Lambda Aliases

A Lambda alias is a named pointer to a published function version. Aliases help applications call a stable name such as `prod` while the team changes which version receives traffic.

## Why use aliases?

Without aliases, callers may invoke a numbered version or `$LATEST` directly. With aliases, the application invokes a stable qualifier:

```text
demo-lambda:prod
```

The alias can be moved from version `4` to version `5` without changing the caller's function name.

## Create an alias

```bash
aws lambda create-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 4 \
  --description "Production alias"
```

## Update an alias

```bash
aws lambda update-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 5
```

## Invoke through an alias

```bash
aws lambda invoke \
  --function-name demo-lambda:prod \
  --payload '{}' \
  response.json

cat response.json
```

## Weighted routing

An alias can split traffic between its primary version and one additional version. Example: send 90% to version 4 and 10% to version 5.

```bash
aws lambda update-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 4 \
  --routing-config 'AdditionalVersionWeights={\"5\"=0.1}'
```

The primary version receives the remaining 90%. Weighted routing is useful for gradual rollout, but it does not by itself provide automated rollback or alarm evaluation.

## Inspect aliases

```bash
aws lambda list-aliases --function-name demo-lambda

aws lambda get-alias \
  --function-name demo-lambda \
  --name prod
```

## Best practices

- Use aliases for stable environment entry points.
- Do not point production integrations at `$LATEST`.
- Use separate aliases for environments where appropriate.
- Monitor errors and latency during weighted releases.
- For managed canary/linear deployments and automatic rollback, consider AWS CodeDeploy with Lambda.
