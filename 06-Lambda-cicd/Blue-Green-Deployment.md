# Blue-Green Deployment for AWS Lambda

Blue-green deployment releases a new Lambda version alongside the current version. Traffic is moved to the new version only after validation, with a rollback path to the previous version.

## How it works

```text
                Production callers
                       |
                       v
                  prod alias
                       |
             +---------+---------+
             |                   |
          Version 10          Version 11
           BLUE                GREEN
        Current release      New release
```

The alias initially points to version 10 (blue). After version 11 (green) is tested, move the alias to version 11. If the new release fails, move the alias back to version 10.

## Step 1: Publish the current version

```bash
aws lambda publish-version \
  --function-name demo-lambda \
  --description "Blue release"
```

Record the returned version number. The examples below use version `10`.

## Step 2: Create or set the production alias

```bash
aws lambda create-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 10
```

If the alias already exists, use `update-alias` instead.

## Step 3: Deploy and publish the green version

Update the function code, wait for the update to finish, run validation, then publish a new version:

```bash
aws lambda update-function-code \
  --function-name demo-lambda \
  --zip-file fileb://function.zip

aws lambda wait function-updated \
  --function-name demo-lambda

aws lambda publish-version \
  --function-name demo-lambda \
  --description "Green release"
```

Assume the new version is `11`.

## Step 4: Test the green version directly

```bash
aws lambda invoke \
  --function-name demo-lambda:11 \
  --payload '{}' \
  green-response.json

cat green-response.json
```

Use representative test events and validate dependencies, permissions, and downstream behavior.

## Step 5: Shift production traffic

For a simple all-at-once cutover:

```bash
aws lambda update-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 11
```

For a gradual rollout, configure weighted alias routing or use CodeDeploy deployment preferences such as canary or linear traffic shifting. CodeDeploy can use CloudWatch alarms to stop or roll back a deployment.

## Step 6: Roll back if required

```bash
aws lambda update-alias \
  --function-name demo-lambda \
  --name prod \
  --function-version 10
```

Rollback is quick because the previous published version remains available. Ensure that database changes and external side effects are backward-compatible; moving an alias back cannot automatically reverse data migrations or messages already processed.

## Production checklist

- [ ] New version is published and directly tested.
- [ ] Alias is used by production integrations.
- [ ] CloudWatch alarms cover errors, duration, and throttles.
- [ ] Logs and application metrics are checked after traffic moves.
- [ ] Rollback version is retained.
- [ ] Database and event-schema changes are backward-compatible.
- [ ] Deployment role has only the permissions it needs.

## Blue-green vs canary

| Strategy | Traffic behavior | When to use |
|---|---|---|
| Blue-green | Switch all traffic from old to new version | Fast cutover with a clear rollback target |
| Canary | Send a small percentage first, then increase | Reduce risk while validating real traffic |
| Linear | Increase traffic in fixed steps | Gradual rollout over a defined period |

## Key point

Lambda versions provide immutable releases, aliases provide stable routing, and CodeDeploy can automate gradual traffic shifting and alarm-based rollback.
