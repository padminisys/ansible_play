# Fail2Ban Ansible Collection

[![Galaxy Version](https://img.shields.io/badge/galaxy-security.fail2ban-blue.svg)](https://galaxy.ansible.com/security/fail2ban)
[![License](https://img.shields.io/badge/license-Apache%202.0-brightgreen.svg)](LICENSE)
[![CI Status](https://github.com/company/ansible-fail2ban-collection/workflows/CI/badge.svg)](https://github.com/company/ansible-fail2ban-collection/actions)

An Ansible collection for installing, configuring, and managing fail2ban on CentOS/RHEL systems. This collection is designed for AWX/Ansible Tower integration and follows security best practices.

## Features

- ✅ **CentOS 10 Stream Support**: Optimized for CentOS 10 Stream and RHEL 9+
- ✅ **AWX Integration**: Ready-to-use with AWX/Ansible Tower
- ✅ **Security Hardened**: Follows CIS benchmarks and security best practices
- ✅ **Idempotent**: All tasks are fully idempotent and safe to run multiple times
- ✅ **Comprehensive Jails**: Pre-configured jails for SSH, HTTP, and more
- ✅ **Environment-Specific**: Different configurations for dev/staging/production
- ✅ **Monitoring Ready**: Built-in validation and status checking
- ✅ **Email Notifications**: Configurable email alerts for security events

## Quick Start

### 1. Install the Collection

```bash
ansible-galaxy collection install security.fail2ban
```

### 2. Basic Usage

```yaml
---
- name: Install fail2ban
  hosts: all
  become: true
  tasks:
    - name: Include fail2ban playbook
      ansible.builtin.include: security.fail2ban.install_fail2ban
```

## AWX Integration Guide

### Prerequisites

1. **AWX/Ansible Tower** 4.0+ installed and configured
2. **CentOS 10 Stream** or **RHEL 9+** target systems
3. **Sudo access** on target systems
4. **GitHub repository** with this collection

### Step 1: Create AWX Project

1. Navigate to **Projects** in AWX
2. Click **Add** to create a new project
3. Configure the project:
   - **Name**: `Fail2Ban Security Collection`
   - **Organization**: Your organization
   - **SCM Type**: `Git`
   - **SCM URL**: `https://github.com/your-org/ansible-fail2ban-collection.git`
   - **SCM Branch/Tag/Commit**: `main`
   - **SCM Update Options**: 
     - ✅ Clean
     - ✅ Delete on Update
     - ✅ Update Revision on Launch
   - **Cache Timeout**: `0` (for development) or `300` (for production)

### Step 2: Create Inventory

1. Navigate to **Inventories** in AWX
2. Click **Add** → **Add Inventory**
3. Configure the inventory:
   - **Name**: `CentOS Servers`
   - **Organization**: Your organization
   - **Variables**: (Optional, can be set per host/group)

#### Add Hosts to Inventory

1. Click on your inventory
2. Go to **Hosts** tab
3. Click **Add** to add hosts
4. For each host, configure:
   - **Host Name**: `server-name`
   - **Variables**:
     ```yaml
     ansible_host: 10.0.1.10
     ansible_user: ansible
     ansible_become: true
     ```

#### Create Host Groups (Optional)

1. Go to **Groups** tab in your inventory
2. Create groups like:
   - `production`
   - `staging` 
   - `webservers`
   - `dbservers`

### Step 3: Create Credentials

1. Navigate to **Credentials** in AWX
2. Click **Add** to create credentials
3. Configure machine credentials:
   - **Name**: `CentOS SSH Key`
   - **Credential Type**: `Machine`
   - **Username**: `ansible`
   - **SSH Private Key**: (paste your private key)
   - **Privilege Escalation Method**: `sudo`

### Step 4: Create Job Template

1. Navigate to **Templates** in AWX
2. Click **Add** → **Add Job Template**
3. Configure the job template:
   - **Name**: `Install Fail2Ban`
   - **Job Type**: `Run`
   - **Inventory**: `CentOS Servers`
   - **Project**: `Fail2Ban Security Collection`
   - **Playbook**: `playbooks/install_fail2ban.yml`
   - **Credentials**: `CentOS SSH Key`
   - **Variables**: (See variables section below)
   - **Options**:
     - ✅ Enable Privilege Escalation
     - ✅ Enable Fact Storage
     - ✅ Enable Concurrent Jobs

#### Job Template Variables

```yaml
---
# Environment-specific settings
fail2ban_default_bantime: 3600
fail2ban_default_maxretry: 5
fail2ban_destemail: security@company.com

# Jails configuration
fail2ban_jails:
  - name: sshd
    enabled: true
    port: ssh
    filter: sshd
    logpath: /var/log/secure
    maxretry: 3
    bantime: 3600
    findtime: 600

# Whitelist your management IPs
fail2ban_ignoreip:
  - 127.0.0.1/8
  - ::1
  - 10.0.0.0/24  # Your management network
```

### Step 5: Create Workflow Template (Advanced)

For complex deployments, create a workflow template:

1. Navigate to **Templates** in AWX
2. Click **Add** → **Add Workflow Template**
3. Configure:
   - **Name**: `Security Hardening Workflow`
   - **Organization**: Your organization
   - **Inventory**: `CentOS Servers`

4. Add workflow nodes:
   - **Node 1**: System Updates
   - **Node 2**: Install Fail2Ban (this collection)
   - **Node 3**: Configure Firewall
   - **Node 4**: Security Validation

## Configuration Variables

### Global Variables (group_vars/all.yml)

| Variable | Default | Description |
|----------|---------|-------------|
| `fail2ban_default_bantime` | `3600` | Default ban time in seconds |
| `fail2ban_default_findtime` | `600` | Time window for counting failures |
| `fail2ban_default_maxretry` | `5` | Maximum retry attempts before ban |
| `fail2ban_loglevel` | `INFO` | Logging level (DEBUG, INFO, WARNING, ERROR) |
| `fail2ban_destemail` | `root@localhost` | Email for notifications |
| `fail2ban_ignoreip` | `['127.0.0.1/8', '::1']` | IP addresses to whitelist |

### Environment-Specific Variables

#### Production (group_vars/production.yml)
- Stricter security settings
- Longer ban times
- Email notifications enabled
- Enhanced logging

#### Staging (group_vars/staging.yml)
- Moderate security settings
- Standard ban times
- Development team notifications

#### Development (group_vars/development.yml)
- Lenient security settings
- Short ban times
- Developer-friendly configuration

### Jail Configuration

Configure jails using the `fail2ban_jails` variable:

```yaml
fail2ban_jails:
  - name: sshd
    enabled: true
    port: ssh
    filter: sshd
    logpath: /var/log/secure
    maxretry: 3
    bantime: 3600
    findtime: 600
  - name: httpd-auth
    enabled: true
    port: http,https
    filter: apache-auth
    logpath: /var/log/httpd/error_log
    maxretry: 6
    bantime: 1800
```

## Security Considerations

### 1. Credential Management
- Use AWX's encrypted credential store
- Never store passwords in plain text
- Rotate SSH keys regularly
- Use separate credentials per environment

### 2. Network Security
- Always whitelist your management networks in `fail2ban_ignoreip`
- Test ban rules in staging before production
- Monitor fail2ban logs for false positives
- Configure firewall integration properly

### 3. Access Control
- Use AWX RBAC to control who can run jobs
- Limit job template permissions
- Audit job execution logs
- Implement approval workflows for production

## Monitoring and Validation

The playbook includes comprehensive validation tasks:

1. **Service Status Check**: Verifies fail2ban service is running
2. **Jail Status Check**: Confirms all enabled jails are active
3. **Configuration Validation**: Ensures configuration files are valid
4. **Log File Check**: Verifies log files are being written

### Manual Validation Commands

```bash
# Check fail2ban status
sudo fail2ban-client status

# Check specific jail
sudo fail2ban-client status sshd

# View banned IPs
sudo fail2ban-client get sshd banip

# Check logs
sudo tail -f /var/log/fail2ban.log
```

## Troubleshooting

### Common Issues

1. **Service fails to start**
   - Check configuration syntax: `fail2ban-client -t`
   - Review logs: `journalctl -u fail2ban`
   - Verify log file permissions

2. **Jails not starting**
   - Check log file paths exist
   - Verify filter files are present
   - Ensure backend is compatible (systemd vs polling)

3. **False positives**
   - Add legitimate IPs to `fail2ban_ignoreip`
   - Adjust `maxretry` and `findtime` values
   - Review filter regex patterns

### AWX-Specific Issues

1. **Job fails with permission errors**
   - Verify sudo configuration on target hosts
   - Check SSH key permissions
   - Ensure `ansible_become: true` is set

2. **Inventory sync issues**
   - Check GitHub repository access
   - Verify branch/tag names
   - Review project sync logs

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests: `molecule test`
5. Submit a pull request

## License

Apache License 2.0 - see [LICENSE](LICENSE) file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/company/ansible-fail2ban-collection/issues)
- **Documentation**: [GitHub Wiki](https://github.com/company/ansible-fail2ban-collection/wiki)
- **Security**: security@company.com

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and changes.