# Security Policy

## Supported Versions
This repository is a learning/portfolio project. Only the `main` branch is actively maintained.

## Reporting a Vulnerability
If you believe you found a security issue:

1. **Do not open a public issue** with sensitive details.
2. Send a report via a private channel (e.g., email / direct message) including:
   - summary of the issue
   - steps to reproduce
   - potential impact
   - any suggested fixes (if you have them)

## Sensitive Data
Please do **not** submit:
- credentials, passwords, MFA secrets, scratch codes
- private SSH keys
- cloud access keys
- files from `/root/bootstrap-secrets/` or similar

If you accidentally committed secrets, rotate them immediately and open a PR that removes them from the repo history (if needed).