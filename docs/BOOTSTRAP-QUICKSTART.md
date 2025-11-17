# Bootstrap Playbook - Quick Start Guide

## Setup (One-Time)

### 1. Create Vault File
```bash
cd /opt/ansible/inventory/group_vars
cp vault-example.yml vault.yml
vim vault.yml  # Add your actual values
ansible-vault encrypt vault.yml
```

### 2. Add Servers to Inventory
```bash
vim /opt/ansible/inventory/hosts
```

## Common Commands

### Full Bootstrap
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit <hostname> \
  --ask-vault-pass
```

### Skip Domain Join
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit <hostname> \
  --skip-tags domain-join \
  --ask-vault-pass
```

### Skip Reboot
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit <hostname> \
  --skip-tags reboot \
  --ask-vault-pass
```

### Multiple Exclusions
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit <hostname> \
  --skip-tags "domain-join,reboot" \
  --ask-vault-pass
```

### Verbose Output
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit <hostname> \
  --ask-vault-pass \
  -vvv
```

## Available Tags

Skip any role using `--skip-tags`:
- `hostname` - Set hostname
- `satellite` - Register to Satellite
- `packages` - Update packages
- `selinux` - Configure SELinux
- `ssh-config` - Deploy SSH key
- `sudo` - Configure sudo
- `ssh-hardening` - Disable root SSH
- `domain-join` - Join AD domain
- `firewall` - Configure firewall
- `auditd` - Configure auditing
- `reboot` - Reboot server

## Quick Checks

### Verify Vault Variables
```bash
ansible-vault view /opt/ansible/inventory/group_vars/vault.yml
```

### Test Connectivity
```bash
ansible <hostname> -m ping
```

### Check Role Syntax
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml --syntax-check
```

### Dry Run (Check Mode)
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit <hostname> \
  --check \
  --ask-vault-pass
```

## Report Location
```
/opt/ansible/reports/bootstrap_report_<timestamp>.txt
```

## For Full Documentation
See: `/opt/ansible/playbooks/BOOTSTRAP-README.md`
