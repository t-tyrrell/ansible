# Bootstrap Playbook Documentation

## Overview
The bootstrap playbook configures new Linux servers with base security and operational settings required for the northshore.local environment.

## What It Does

### Roles Executed (in order):
1. **hostname** - Sets hostname and FQDN from inventory
2. **satellite** - Registers to Red Hat Satellite (deathstar)
3. **packages** - Updates all packages and installs base tools (vim)
4. **selinux** - Ensures SELinux is in enforcing mode
5. **ssh-config** - Deploys service-satellite SSH key
6. **sudo** - Configures passwordless sudo for g_linux AD group
7. **ssh-hardening** - Disables root SSH login
8. **domain-join** - Joins server to northshore.local AD domain
9. **firewall** - Configures base firewall with SSH access rules
10. **auditd** - Installs and configures system auditing
11. **reboot** - Conditionally reboots if needed

## Prerequisites

### 1. Ansible Vault Setup
Create an encrypted vault file with sensitive credentials:

```bash
cd /opt/ansible/inventory/group_vars
cp vault-example.yml vault.yml
# Edit vault.yml with your actual values
ansible-vault encrypt vault.yml
```

Required vault variables:
- `vault_ssh_service_satellite_pubkey` - SSH public key for service-satellite
- `vault_ad_join_username` - AD service account for domain joins
- `vault_ad_join_password` - Password for AD service account

### 2. Inventory Configuration
Add new servers to `/opt/ansible/inventory/hosts`:

```ini
[new-servers]
newserver01
newserver02
```

### 3. Satellite Activation Keys
Ensure these activation keys exist in Satellite:
- `RHEL8_CORE` - For RHEL 8 servers
- `RHEL9_CORE` - For RHEL 9 servers

## Usage

### Bootstrap All Servers in a Group
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit new-servers \
  --ask-vault-pass
```

### Bootstrap a Single Server
```bash
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit newserver01 \
  --ask-vault-pass
```

### Skip Specific Roles
```bash
# Skip domain join
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit new-servers \
  --skip-tags domain-join \
  --ask-vault-pass

# Skip reboot
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit new-servers \
  --skip-tags reboot \
  --ask-vault-pass
```

### Run Only Specific Roles
```bash
# Only run hostname and satellite registration
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit new-servers \
  --tags "hostname,satellite" \
  --ask-vault-pass
```

### Using Vault Password File
```bash
# Store vault password in a file (secure it properly!)
echo "your_vault_password" > /opt/ansible/.vault_pass
chmod 600 /opt/ansible/.vault_pass

# Run without prompting for password
ansible-playbook /opt/ansible/playbooks/bootstrap.yml \
  --limit new-servers \
  --vault-password-file=/opt/ansible/.vault_pass
```

## Available Tags

Each role can be excluded or run individually using tags:

| Tag | Description |
|-----|-------------|
| `hostname` | Set hostname and FQDN |
| `satellite` | Register to Satellite |
| `packages` | Update packages and install vim |
| `selinux` | Configure SELinux |
| `ssh-config` | Deploy service-satellite SSH key |
| `sudo` | Configure sudo for g_linux |
| `ssh-hardening` | Disable root SSH login |
| `domain-join` | Join Active Directory domain |
| `firewall` | Configure base firewall rules |
| `auditd` | Configure system auditing |
| `reboot` | Conditional reboot |

## Firewall Configuration

### Default SSH Access
The firewall role configures SSH access from these predefined sources:
- 10.1.11.0/24 (Danvers IS)
- 10.1.7.0/24 (VPN)
- 10.1.9.95/32 (Arkanis)
- 10.1.15.46/32 (Satellite/deathstar)
- 10.1.9.236/32 (Fred Jumpbox)
- 10.1.15.50/32 (Barghest)

### Adding Server-Specific Rules
For servers that need additional firewall rules, see the template at:
`/opt/ansible/roles/firewall/templates/additional-rules-template.j2`

Examples include:
- Web servers (HTTP/HTTPS)
- Database servers (MySQL, PostgreSQL, Oracle)
- Application servers (custom ports)
- Monitoring (Prometheus, Zabbix)

## Audit Rules

The auditd role configures comprehensive security auditing:
- Authentication file changes (passwd, shadow, group)
- Sudoers modifications
- SELinux configuration changes
- Kernel module loading
- Time changes
- Network configuration changes
- File deletions by users
- Login/logout events
- Privilege escalation (sudo, su)
- Permission changes (chmod, chown)

## Reports

Bootstrap execution generates a report at:
```
/opt/ansible/reports/bootstrap_report_<timestamp>.txt
```

## Troubleshooting

### Domain Join Fails
- Verify AD credentials in vault file
- Check DNS resolution: `nslookup northshore.local`
- Test domain connectivity: `realm discover northshore.local`
- Ensure required packages are installed: `rpm -qa | grep realmd`

### Satellite Registration Fails
- Verify activation keys exist in Satellite
- Check Satellite connectivity: `ping deathstar.northshore.local`
- Verify katello-ca-consumer package installed

### SSH Access Not Working
- Verify service-satellite public key in vault
- Check authorized_keys: `cat /home/service-satellite/.ssh/authorized_keys`
- Verify SSH service is running: `systemctl status sshd`

### Firewall Blocking Access
- Check firewall rules: `firewall-cmd --list-all`
- Verify rich rules are active: `firewall-cmd --list-rich-rules`
- Test connectivity from allowed sources

## Customization

### Modifying Default Variables
Each role has default variables in `roles/<role_name>/defaults/main.yml`

To override, create host or group variables:
```bash
# For all servers
vim /opt/ansible/inventory/group_vars/all.yml

# For specific group
vim /opt/ansible/inventory/group_vars/new-servers.yml
```

### Example: Different Firewall Rules per Group
```yaml
# In group_vars/web-servers.yml
firewall_additional_rules:
  - service: http
  - service: https
```

## Security Considerations

1. **Vault Password** - Store securely, never commit to git
2. **SSH Keys** - Rotate periodically
3. **AD Credentials** - Use dedicated service account with minimal permissions
4. **Audit Logs** - Monitor /var/log/audit/audit.log regularly
5. **Firewall Rules** - Review and update as needed

## Post-Bootstrap Tasks

After bootstrap completes:
1. Verify all roles executed successfully
2. Test SSH access with service-satellite user
3. Test AD authentication: `su - username@northshore.local`
4. Verify sudo access for g_linux members
5. Check audit logs are being generated
6. Confirm firewall rules are correct

## Support

For issues or questions:
1. Check logs in `/opt/ansible/logs/ansible.log`
2. Review bootstrap report in `/opt/ansible/reports/`
3. Run playbook with verbose output: `-vvv`
