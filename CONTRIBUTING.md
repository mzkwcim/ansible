# Contributing

Thanks for your interest in contributing!

## Scope
This project focuses on:
- Linux bootstrapping/hardening with Ansible
- SSH MFA (TOTP) setup
- diagnostics scripts & troubleshooting runbooks
- CI checks (linting + secret scanning)

## Getting started
1. Fork the repo and create a branch:
   - `feat/<short-name>` for new features
   - `fix/<short-name>` for bug fixes
   - `docs/<short-name>` for documentation changes

2. Keep changes small and focused.

## Local checks
Recommended before opening a PR:

### YAML lint
```bash
yamllint .
```

### Ansible lint
```bash
ansible-lint
```

### Secret scanning (optional locally)
If you have `gitleaks` installed:
```bash
gitleaks detect --source . --no-git
```