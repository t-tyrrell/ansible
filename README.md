

#Northshore Ansible Automation

Infrastructure automation for Northshore Community College.

## Structure
```
/opt/ansible/
├── playbooks/
│   ├── bootstrap/          # Server bootstrap
│   ├── maintenance/        # System maintenance & security
│   └── satellite/          # Satellite content management
├── roles/                  # Reusable roles
├── inventory/              # Hosts and variables
└── docs/                   # Documentation
```

## Quick Start

### Bootstrap new server
```bash
cd /opt/ansible
ansible-playbook playbooks/bootstrap/bootstrap.yml --limit <hostname> --ask-vault-pass
```

### Reboot servers
```bash
ansible-playbook playbooks/maintenance/reboot.yml --limit <hostname>
```

### Satellite content management
```bash
ansible-playbook playbooks/satellite/publish_cv.yml -e "cv_name=RHEL9_CORE"
```

## Roles

**Bootstrap roles:** hostname, satellite, domain_join, sudo, ssh_hardening, selinux, firewall, auditd, packages, reboot

**Satellite roles:** satellite_publish, satellite_promote_nonprod, satellite_promote_prod

## Documentation

See `docs/` directory for detailed guides.
