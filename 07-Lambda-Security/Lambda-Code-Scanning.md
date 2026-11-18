# AWS Lambda Code Scanning

Code scanning helps identify vulnerabilities, exposed secrets, unsafe coding patterns, and vulnerable dependencies before a Lambda function is deployed.

## Suggested security checks

| Check | Example tools |
|---|---|
| Secret detection | Gitleaks, TruffleHog |
| Static application security testing (SAST) | Semgrep, SonarQube |
| Dependency scanning | pip-audit, Dependabot, Snyk |
| Infrastructure-as-code scanning | Checkov, tfsec |
| ZIP / filesystem scanning | Trivy |
| Runtime monitoring | CloudWatch, AWS Security services |

Choose tools based on your language, repository, risk model, and licensing requirements.

## Example Python dependency scan

```bash
python -m pip install pip-audit
pip-audit -r requirements.txt
```

## Example secret scan with Gitleaks

```bash
gitleaks detect \
  --source . \
  --redact \
  --no-git
```

Check the installed Gitleaks version and command options; flags may vary by release.

## Example Semgrep scan

```bash
semgrep scan \
  --config auto \
  --error \
  .
```

Review findings rather than blindly suppressing alerts. Add documented exclusions only when justified.

## Example GitHub Actions security stage

```yaml
name: Lambda Security Checks

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependency scanner
        run: python -m pip install pip-audit

      - name: Scan dependencies
        run: pip-audit -r requirements.txt

      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: auto
```

For production, review and pin third-party actions to trusted full commit SHAs, and use your organization's approved action sources.

## CI/CD security gates

A practical pipeline can use this order:

```text
Checkout
   |
Secret Scan
   |
SAST + Dependency Scan
   |
Unit Tests
   |
Package Lambda Artifact
   |
Artifact / IaC Scan
   |
Deploy to Test
   |
Smoke Tests
   |
Production Approval and Deploy
```

## Best practices

- Scan every pull request and the default branch.
- Fail builds for defined high-severity findings, with a documented exception process.
- Keep scanners and vulnerability databases current.
- Generate and retain an SBOM for traceability where useful.
- Scan dependencies and rebuild when new vulnerabilities are disclosed.
- Do not upload secrets or production data to scanning services without approval.
- Treat a clean scan as one control, not proof that the function is secure.
