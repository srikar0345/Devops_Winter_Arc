# Day 13 — Docker Security & Optimization

## Objective

Build an intentionally unsafe Docker image, scan it with Trivy,
remediate security issues, and verify a hardened runtime.

## Security improvements

- Replaced `python:latest` with `python:3.12-slim`.
- Used a multi-stage build to separate dependency installation
  from the runtime image.
- Removed unnecessary build tools from the final image.
- Created a dedicated application user with UID 10001.
- Removed the hard-coded API token from Dockerfile.good.
- Passed a fake training token at runtime.
- Used exec-form CMD.
- Applied CPU, memory and PID limits.
- Enabled a read-only root filesystem and temporary storage.
- Dropped Linux capabilities and enabled no-new-privileges.

## Before and after comparison

| Metric | Before: my-app:bad | After: my-app:v2 |
|---|---:|---:|
| Docker content size | 458 MB | 44.9 MB |
| Docker disk usage | 1.76 GB | 185 MB |
| HIGH findings | 401 | 44 |
| CRITICAL findings | 16 | 0 |
| Runtime user | Root by default | app, UID 10001 |
| Hard-coded token | Present | Removed |

The image content size decreased by approximately 90.2%.
HIGH findings decreased by approximately 89.0%, and CRITICAL
findings decreased from 16 to 0.

## Runtime verification

- GET / returned the expected JSON response.
- GET /health returned `ok`.
- The image's configured user is `app`.
- Runtime checks confirmed:
  - Read-only root filesystem
  - Memory limit of 256 MiB
  - CPU limit of 0.5 CPU
  - PID limit of 100
  - All Linux capabilities dropped
  - no-new-privileges enabled

## Trivy scanning

Trivy version: 0.76.0

Before report: reports/before.json
After report: reports/after.json

Commands used:

```bash
trivy image --severity HIGH,CRITICAL \
  -f json -o reports/before.json my-app:bad

trivy image --severity HIGH,CRITICAL \
  -f json -o reports/after.json my-app:v2

trivy config .
```

## Remaining work

The after scan still reports 44 HIGH findings. Review the
individual vulnerabilities, available fixes, and base-image
updates before considering the image ready for production.

A clean or improved scan alone does not guarantee security.
Environment variables were used for this training token; use
a secrets manager or an appropriate secret-mounting mechanism
for production credentials.

## Evidence

See the `evidence/` directory for screenshots of scans,
image sizes, non-root execution, and runtime restrictions.
