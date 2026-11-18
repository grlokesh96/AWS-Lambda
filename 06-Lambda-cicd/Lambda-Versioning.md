# AWS Lambda Versioning

Lambda versioning lets you publish an immutable snapshot of a function's code and most version-specific configuration.

## `$LATEST` and published versions

- `$LATEST` is the mutable working version.
- A published version is a numbered, immutable snapshot such as `1`, `2`, or `3`.
- Updating function code or configuration changes `$LATEST`; it does not modify an already published version.
- A published version can be invoked directly or through an alias.

## Enable versioning by publishing

```bash
aws lambda publish-version \
  --function-name demo-lambda \
  --description "Release 1"
```

The command returns a version number and ARN.

List versions:

```bash
aws lambda list-versions-by-function \
  --function-name demo-lambda
```

Invoke a specific version:

```bash
aws lambda invoke \
  --function-name demo-lambda:1 \
  --payload '{}' \
  response.json
```

## Update and publish a new version

```bash
aws lambda update-function-code \
  --function-name demo-lambda \
  --zip-file fileb://function.zip

aws lambda wait function-updated \
  --function-name demo-lambda

aws lambda publish-version \
  --function-name demo-lambda \
  --description "Release 2"
```

## Versioning best practices

- Publish only after tests and deployment validation pass.
- Use descriptive version descriptions or release metadata.
- Use aliases such as `dev`, `staging`, and `prod` to avoid hardcoding version numbers in clients.
- Retain versions required for rollback and remove obsolete versions according to a retention policy.
- Remember that version-specific settings are captured when publishing; environment variables and other configuration changes require publishing a new version to be reflected in that version.

## Important distinction

A version is an immutable release snapshot. An alias is a movable pointer to a version. Use both together for a manageable release process.
