# Ops Bootstrap: Linux Hardening + SSH MFA (TOTP) + Diagnostics (Ansible)

A minimal, portfolio-ready Linux server “bootstrap” that:
- creates an admin user and configures SSH key access,
- (optionally) sets a strong password,
- enables SSH MFA (Google Authenticator / TOTP) for a selected user,
- hardens SSH (root off, password auth off — after verification),
- stores MFA secrets (TOTP secret / otpauth URL / scratch codes) in a **root-only** file,
- is structured to be easily extended with fail2ban/UFW/Docker and troubleshooting runbooks.

> Goal: demonstrate “Linux terminal skills + security mindset + automation + idempotent provisioning”.

---

## What this does

### Bootstrap (Ansible)
- ✅ creates `admin_user` and adds it to `sudo`
- ✅ adds an SSH public key (`admin_ssh_pubkey`)
- ✅ generates a strong password (optional)
- ✅ enables TOTP MFA for SSH via PAM Google Authenticator
- ✅ enforces MFA only for `admin_user` (via `Match User` block in `sshd_config`)
- ✅ saves secrets to: `/root/bootstrap-secrets/<host>-<user>.txt` (0600)
- ✅ restarts SSH when configuration changes
---

## Requirements

- Linux: Ubuntu/Debian (tested on Ubuntu 22.04/24.04)
- SSH access to the host (root or a sudo-enabled user)
- Python on the host (usually already installed)
- On your local machine:
  - Ansible 2.15+ (or newer)
  - SSH key for the target host

---

## Repository layout

```text
ansible/
  inventories/hosts.yml
  playbooks/bootstrap.yml
  roles/bootstrap_mfa_ssh/
    defaults/main.yml
    tasks/main.yml
    handlers/main.yml
docs/
  runbooks/
scripts/
reports/            
```

---

## Quick Start

### 1.Configure inventory
Edit `ansible/inventories/hosts.yml`:
```yaml
all:
  hosts:
    myvm:
      ansible_host: 1.2.3.4
      ansible_user: ubuntu
      ansible_port: 22
      ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```
---
### 2.Configure role defaults
Edit `ansible/roles/bootstrap_mfa_ssh/defaults/main.yml`.
Key settings:
* admin_user
* admin_ssh_pubkey
* set_strong_password (true/false)
* enable_mfa (true/false)
* disable_password_auth (recommended false for the first run, then true)

---

### 3.Run bootstrap
```bash
cd ansible
ansible-playbook -i inventories/hosts.yml playbooks/bootstrap.yml
```
After the playbook completes:

secrets are stored on the host at:
`/root/bootstrap-secrets/<host>-<user>.txt` 

---
### Safe rollout (avoid locking yourself out)
1. First run:
    * set `disable_password_auth`: false
    * make sure `admin_ssh_pubkey` is configured

2. Configure MFA:
    * copy the `otpauth` URL (or `MFA secret`) from the secrets file
    * add it to your TOTP app (Google Authenticator / Authy / 1Password, etc.)

3. Test in a new SSH session:
    * keep the current session open
    * SSH again as `admin_user` and confirm MFA works

4. Only then set:
    * `disable_password_auth`: true
    * run the playbook again

---
### What SSH login looks like (with MFA)
After `ssh admin_user@host`:
* SSH public key authentication happens first,
* then you’ll be prompted for a 6-digit TOTP code.

If you lose access to your authenticator app, use the scratch codes stored in the secrets file.

## Ops notes / troubleshooting
### Service status
```bash
sudo systemctl status ssh
sudo systemctl status fail2ban   
```

### SSH logs
```bash
sudo journalctl -u ssh -e
```

### Config verification
```bash
sudo sshd -t
sudo grep -n "AuthenticationMethods" /etc/ssh/sshd_config
sudo grep -n "pam_google_authenticator" /etc/pam.d/sshd
```

---
### Security & secrets
* Never commit secrets (passwords, TOTP secrets, scratch codes) to git.
* Secrets are stored on the host under:
    * `/root/bootstrap-secrets` (0700)
    * secret files (0600)

Recommended:
* add `reports/` to `.gitignore`
* only paste non-sensitive outputs into README (e.g., “SSH restarted OK”, “MFA enabled”).

## Roadmap (next steps)
- [ ] Fail2ban + sshd jail
- [x] UFW/firewalld minimal policies
- [ ] Docker + docker compose plugin
- [ ] scripts/diag.sh: host health report (CPU/RAM/DISK/PORTS/DNS/HTTP)
- [ ] CI: ansible-lint + yamllint + gitleaks
- [ ] Terraform: AWS VPC + EC2 + IAM + S3 (separate module/repo)