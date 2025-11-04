# Bootstrap RHEL Server Playbook

This playbook automates the bootstrap process for new RHEL servers with the following steps:

1. Set hostname
2. Join Active Directory using realm
3. Configure realm access control for AD groups
4. Add sudoers configuration for g_linux group
5. Register with Red Hat Satellite using activation keys
6. Run system updates
7. Install CrowdStrike Falcon
8. Remove root SSH access
9. Configure firewall rules

## Prerequisites

- Ansible control node with the following collections installed:
  - `ansible.posix`
  - `community.general`
- Target server must be accessible via SSH
- Target server must have network connectivity to:
  - Active Directory domain controllers
  - Red Hat Satellite server
  - CrowdStrike download location (if applicable)

## Initial Setup

### 1. Create the Vault File

Copy the template and add your sensitive passwords:

```bash
cd /home/ttyrrell/ansible-work
cp group_vars/bootstrap_vault.yml.template group_vars/bootstrap_vault.yml
```

Edit the file and add your passwords:

```bash
vi group_vars/bootstrap_vault.yml
```

Update the following values:
- `ad_join_password`: Password for your AD service account
- `vault_crowdstrike_cid`: CrowdStrike Customer ID (if using CrowdStrike)

### 2. Encrypt the Vault File

```bash
ansible-vault encrypt group_vars/bootstrap_vault.yml
```

You'll be prompted to create a vault password. **Remember this password!**

### 3. Update the Variables File (Optional)

Edit `group_vars/bootstrap.yml` to customize:
- AD domain and OU configuration
- Satellite server details
- Firewall rules
- CrowdStrike settings

## Running the Playbook

### Basic Usage

```bash
ansible-playbook playbooks/bootstrap-rhel-server.yml \
  --ask-vault-pass \
  -e bootstrap_hostname=newserver \
  -l newserver.northshore.local
```

When prompted:
1. Enter your Ansible Vault password
2. Select the Satellite activation key (1-3):
   - 1) rhel8-nonprod
   - 2) rhel8-prod
   - 3) rhel9-nonprod

### Run Specific Tasks with Tags

You can run only specific sections using tags:

```bash
# Only set hostname
ansible-playbook playbooks/bootstrap-rhel-server.yml --ask-vault-pass --tags hostname

# Only join AD and configure access
ansible-playbook playbooks/bootstrap-rhel-server.yml --ask-vault-pass --tags ad_join,ad_access

# Only register with Satellite
ansible-playbook playbooks/bootstrap-rhel-server.yml --ask-vault-pass --tags satellite

# Only configure firewall
ansible-playbook playbooks/bootstrap-rhel-server.yml --ask-vault-pass --tags firewall

# Skip updates
ansible-playbook playbooks/bootstrap-rhel-server.yml --ask-vault-pass --skip-tags updates
```

Available tags:
- `hostname` - Set system hostname
- `ad_join` - Join Active Directory
- `ad_access` - Configure AD access control
- `sudoers` - Configure sudoers for g_linux group
- `satellite` - Register with Satellite
- `updates` - Run system updates
- `crowdstrike` - Install CrowdStrike
- `ssh` - Configure SSH (disable root)
- `firewall` - Configure firewall rules

### Dry Run (Check Mode)

Test what would change without making actual changes:

```bash
ansible-playbook playbooks/bootstrap-rhel-server.yml \
  --ask-vault-pass \
  -e bootstrap_hostname=newserver \
  -l newserver.northshore.local \
  --check
```

## Managing the Vault

### View Vault Contents

```bash
ansible-vault view group_vars/bootstrap_vault.yml
```

### Edit Vault Contents

```bash
ansible-vault edit group_vars/bootstrap_vault.yml
```

This is useful when your AD password changes.

### Change Vault Password

```bash
ansible-vault rekey group_vars/bootstrap_vault.yml
```

## Troubleshooting

### AD Join Fails

Check:
- Network connectivity to domain controllers
- AD credentials in vault are correct
- Computer OU exists in AD
- DNS resolution is working

### Satellite Registration Fails

Check:
- Network connectivity to Satellite server
- Activation key is valid and has content views assigned
- Organization name matches exactly

### Firewall Rules Not Applied

Verify firewalld is installed and running:
```bash
systemctl status firewalld
```

### SSH Connection Lost After Running

The playbook disables root SSH and restricts SSH to specific networks. Ensure:
- Your source IP is in the allowed networks list in `bootstrap.yml`
- You can login with an AD account from g_linux group after the playbook completes

## Post-Bootstrap Steps

After running the bootstrap playbook:

1. **Verify AD join**: Test logging in with an AD account
2. **Verify Satellite registration**: Check the server appears in Satellite
3. **Verify firewall rules**: Test SSH access from allowed networks
4. **Consider rebooting**: If kernel updates were applied

## Example: Complete Bootstrap

```bash
# 1. Ensure server is in inventory
echo "testserver01" >> hosts

# 2. Run bootstrap playbook
ansible-playbook playbooks/bootstrap-rhel-server.yml \
  --ask-vault-pass \
  -e bootstrap_hostname=testserver01 \
  -l testserver01.northshore.local

# 3. Select activation key when prompted (e.g., 1 for rhel8-nonprod)

# 4. Wait for completion and review output

# 5. Test AD login
ssh user@northshore.local@testserver01.northshore.local
```

## Security Notes

- The vault file contains sensitive passwords - protect your vault password
- Root SSH access is disabled after bootstrap completes
- SSH is restricted to specific source networks only
- All passwords are marked with `no_log: true` to prevent logging
- Consider using a dedicated service account for AD joins with minimal privileges
