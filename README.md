# validate-certificates

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![GitHub Release](https://img.shields.io/github/v/release/AbsaOSS/validate-certificates)](https://github.com/AbsaOSS/validate-certificates/releases)

A GitHub composite action that validates **multiple** SSL/TLS certificates provided as a JSON array and produces a consolidated summary table in the job log and the GitHub Step Summary panel.

Certificates are **grouped by subject**, so an expired cert with a newer valid replacement for the same subject does not fail the check — only truly unrecoverable expirations cause a failure.

---

## Features

- Accepts any number of certificates as a JSON array of file paths
- Parses subject, issuer, and validity dates using `openssl`
- Works on **Linux** (GNU date) and **macOS** (BSD date) runners
- Groups certificates by subject — expired certs with a valid replacement are downgraded to informational
- Emits `::warning` annotations in the Actions log for actionable issues
- Writes a rich Markdown table to the **GitHub Step Summary** panel
- Exits **1** only when there is an unrecoverable failure (no valid replacement exists)

---

## Inputs

| Input          | Required | Default | Description                                             |
|----------------|----------|---------|---------------------------------------------------------|
| `certificates` | ✅       | —       | JSON array of certificate file paths to validate        |
| `warning_days` | ❌       | `30`    | Days before expiry to emit a warning instead of passing |
| `fail_on_warn` | ❌       | `false` | Fail the job (exit 1) when any certificate is nearing expiry |

---

## Outputs

| Output       | Description                                                            |
|--------------|------------------------------------------------------------------------|
| `result`     | Overall validation result: `pass`, `warn`, or `fail`                   |
| `fail_count` | Number of certificates that failed validation with no valid replacement |
| `warn_count` | Number of certificates nearing expiry with no newer replacement        |

The action also writes:
- Per-certificate validation details to the job log
- A Markdown summary table to the **GitHub Step Summary** panel
- Exits **1** if any certificate is expired, not yet valid, or cannot be parsed and has no valid replacement
- Exits **0** (with warnings) if all certs are valid but some are nearing expiry (unless `fail_on_warn: true`)

---

## Status legend

| Icon | Meaning                                                      |
|------|--------------------------------------------------------------|
| ✅   | Valid — expiry is beyond `warning_days`                      |
| ⚠️   | Expiring Soon — expiry within `warning_days`                 |
| ⏳   | Expired (newer replacement exists) — does not fail the check |
| ❌   | Expired / Failed — no valid replacement found                |

---

## Usage

### Recommended: pin to a specific version tag

```yaml
- name: Validate certificates
  uses: AbsaOSS/validate-certificates@v1
  with:
    certificates: '["./certs/server.crt", "./certs/client.pem"]'
    warning_days: '30'
```

### Discover certificates dynamically, then validate

```yaml
jobs:
  find-certs:
    name: Find Certificates
    runs-on: ubuntu-latest
    outputs:
      certs: ${{ steps.set-certificates.outputs.certs }}
    steps:
      - uses: actions/checkout@v4

      - name: Find all .crt and .pem files
        id: set-certificates
        run: |
          certs=$(find ./deployment/certs \
            -type f \( -name "*.crt" -o -name "*.pem" \) \
            | jq -R -s -c 'split("\n")[:-1]')
          echo "certs=$certs" >> $GITHUB_OUTPUT

  validate-certificates:
    name: Validate Certificates
    needs: find-certs
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate all deployment certificates
        uses: AbsaOSS/validate-certificates@v1
        with:
          certificates: ${{ needs.find-certs.outputs.certs }}
          warning_days: '60'
```

### Scheduled certificate health check

```yaml
on:
  schedule:
    - cron: '0 6 * * 1'   # every Monday at 06:00 UTC

jobs:
  cert-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate certificates
        uses: AbsaOSS/validate-certificates@v1
        with:
          certificates: |
            [
              "./certs/api.example.com.crt",
              "./certs/internal.example.com.pem",
              "./certs/legacy.example.com.crt"
            ]
          warning_days: '45'
```

### Using outputs for conditional workflows

```yaml
jobs:
  validate-certs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6

      - name: Validate certificates
        id: cert-check
        uses: AbsaOSS/validate-certificates@v1
        with:
          certificates: '["./certs/server.crt", "./certs/client.pem"]'
          warning_days: '30'

      - name: Send Slack alert on warnings or failures
        if: steps.cert-check.outputs.result != 'pass'
        run: |
          echo "Certificate validation result: ${{ steps.cert-check.outputs.result }}"
          echo "Failures: ${{ steps.cert-check.outputs.fail_count }}"
          echo "Warnings: ${{ steps.cert-check.outputs.warn_count }}"
          # Add your Slack webhook or notification logic here

      - name: Open ticket on failures only
        if: steps.cert-check.outputs.fail_count > 0
        run: |
          echo "Creating incident ticket for ${{ steps.cert-check.outputs.fail_count }} failed certificate(s)"
          # Add your ticketing system integration here
```

### Enforcing strict certificate hygiene

```yaml
- name: Validate certificates with strict policy
  uses: AbsaOSS/validate-certificates@v1
  with:
    certificates: '["./certs/production.crt"]'
    warning_days: '60'
    fail_on_warn: 'true'  # Fail the job if cert expires within 60 days
```

See the [`examples/`](examples/) directory for ready-to-use workflow files.

---

## Requirements

- Runner must have `openssl` and `jq` installed (both are available by default on `ubuntu-latest` and `macos-latest` GitHub-hosted runners)

---

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

---

## License

Copyright 2026 ABSA Group Limited — licensed under the [Apache License 2.0](LICENSE).
