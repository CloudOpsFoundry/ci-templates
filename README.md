# CloudOpsFoundry CI templates

Reusable GitHub Actions workflows for repositories that want consistent CI without copying full pipelines. This repository contains workflow YAML and usage documentation, not application code.

## Python with uv

Call `.github/workflows/python-uv.yml` from a small workflow in your repository. Pin the reference to a reviewed full commit SHA. Private and public repositories can call this public template when their organization policy permits it.

```yaml
name: CI
on: [push, pull_request, workflow_dispatch]
permissions:
  contents: read
jobs:
  validate:
    uses: CloudOpsFoundry/ci-templates/.github/workflows/python-uv.yml@REPLACE_WITH_FULL_COMMIT_SHA
    with:
      python-version: '3.15.2'
      validation-command: uv run --no-sync pytest
```

| Input | Default | Purpose |
| --- | --- | --- |
| `python-version` | Required | Exact Python runtime to install |
| `validation-command` | Required | Repository-owned fail-fast validation command; supports multiple lines |
| `working-directory` | `.` | Project directory within the caller checkout |
| `install-dependencies` | `true` | Install the caller's locked uv dependencies once |
| `all-extras` | `true` | Include every optional dependency extra |
| `workflow-lint` | `true` | Install pinned actionlint and lint workflows |

The caller provides pyproject.toml and uv.lock when dependency installation is enabled. Tests, application-specific validation scripts and release policies stay in the caller repository. Use the same validation command locally. Python is the first supported toolchain; add other focused workflows as real consumers require them.

## Execution and trust

The workflow checks out the **caller repository**, installs checksum-verified uv/actionlint and the requested Python version, installs locked dependencies once, then runs workflow lint and the caller command. Bash stops on errors and pipeline failures. Jobs use Ubuntu hosted runners with a 25-minute limit. No secrets, artifact uploads, deployments, releases or write tokens are configured.

Commands are trusted repository code, not a sandbox for secrets. Keep fork approval and branch protections configured in each caller. The reusable check is named `CI passed`; GitHub may prefix it with the caller job name. Confirm the actual check name before updating required checks. Keep the old required check until the new workflow passes.

Organization allowlists must include the exact reusable workflow reference and the pinned actions it uses. Do not loosen the allowlist to all third-party actions.

## Maintaining templates

Make changes through a reviewed PR and run `Template CI`. Tool versions and archive checksums change together. Tag reviewed versions for discovery, but pin callers to full SHAs for reproducibility. Each caller opts into updates through a PR; updating this repository cannot silently change pinned consumers. Re-run consumer acceptance before releasing incompatible changes.

Licensed under MIT. See [SECURITY.md](SECURITY.md) for private vulnerability reporting.
