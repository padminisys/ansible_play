# AWX Setup Guide for Fail2Ban Collection

This guide provides step-by-step instructions for setting up and using the fail2ban Ansible collection with AWX/Ansible Tower.

## Prerequisites

- AWX/Ansible Tower 4.0+ installed and configured
- CentOS 10 Stream or RHEL 9+ target systems
- GitHub repository containing this collection
- SSH access to target systems with sudo privileges

## Quick Setup Checklist

### 1. Repository Setup
- [ ] Fork or clone this repository to your GitHub organization
- [ ] Ensure the repository is accessible by AWX
- [ ] Configure webhook (optional) for automatic project sync

### 2. AWX Project Configuration
- [ ] Create new project in AWX
- [ ] Set SCM Type to Git
- [ ] Configure repository URL
- [ ] Set branch to `main`
- [ ] Enable SCM update options

### 3. Inventory Setup
- [ ] Create inventory in AWX
- [ ] Add target hosts with proper variables
- [ ] Create host groups (production, staging, etc.)
- [ ] Configure group variables

### 4. Credentials Configuration
- [ ] Create machine credentials for SSH access
- [ ] Configure privilege escalation (sudo)
- [ ] Test connectivity to target hosts

### 5. Job Template Creation
- [ ] Create job template for fail2ban installation
- [ ] Configure playbook path: `playbooks/install_fail2ban.yml`
- [ ] Set appropriate variables
- [ ] Test job execution

## Detailed Configuration

### Project Variables

```yaml
# Basic Configuration
fail2ban_default_bantime: 3600
fail2ban_default_maxretry: 5
fail2ban_destemail: security@company.com

# Jail Configuration
fail2ban_jails:
  - name: sshd
    enabled: true
    port: ssh
    filter: sshd
    logpath: /var/log/secure
    maxretry: 3
    bantime: 3600
    findtime: 600

# IP Whitelist
fail2ban_ignoreip:
  - 127.0.0.1/8
  - ::1
  - 10.0.0.0/24  # Your management network
```

### Environment-Specific Settings

#### Production
```yaml
fail2ban_default_bantime: 7200      # 2 hours
fail2ban_default_maxretry: 3        # Stricter
fail2ban_destemail: security@company.com
```

#### Staging
```yaml
fail2ban_default_bantime: 1800      # 30 minutes
fail2ban_default_maxretry: 5        # Standard
fail2ban_destemail: devops@company.com
```

#### Development
```yaml
fail2ban_default_bantime: 600       # 10 minutes
fail2ban_default_maxretry: 10       # Lenient
fail2ban_destemail: dev@company.com
```

## Job Template Examples

### Basic Installation Job
- **Name**: Install Fail2Ban
- **Playbook**: `playbooks/install_fail2ban.yml`
- **Inventory**: Your server inventory
- **Limit**: Leave blank for all hosts or specify group

### Production Deployment Job
- **Name**: Deploy Fail2Ban - Production
- **Playbook**: `playbooks/install_fail2ban.yml`
- **Inventory**: Production Servers
- **Limit**: `production`
- **Variables**: Production-specific settings

## Workflow Templates

Create workflow templates for complex deployments:

1. **Security Hardening Workflow**
   - Node 1: System Updates
   - Node 2: Install Fail2Ban
   - Node 3: Configure Firewall
   - Node 4: Security Validation

2. **Server Provisioning Workflow**
   - Node 1: Base System Setup
   - Node 2: Security Tools (Fail2Ban)
   - Node 3: Application Deployment
   - Node 4: Monitoring Setup

## Monitoring and Validation

### Built-in Validation
The playbook includes automatic validation:
- Service status verification
- Jail status checking
- Configuration file validation
- Log file accessibility

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

### AWX Job Monitoring
- Monitor job execution in AWX dashboard
- Review job output for validation results
- Set up notifications for job failures
- Use job templates for regular validation

## Security Best Practices

### Credential Management
- Use AWX's encrypted credential store
- Rotate SSH keys regularly
- Use separate credentials per environment
- Never store passwords in plain text

### Network Security
- Always whitelist management networks
- Test ban rules in staging first
- Monitor for false positives
- Configure firewall integration

### Access Control
- Use AWX RBAC features
- Limit job template permissions
- Implement approval workflows for production
- Audit job execution logs

## Troubleshooting

### Common Issues

1. **Service fails to start**
   ```bash
   # Check configuration
   sudo fail2ban-client -t
   
   # Check logs
   sudo journalctl -u fail2ban
   ```

2. **Jails not starting**
   ```bash
   # Verify log file paths
   ls -la /var/log/secure
   
   # Check filter files
   ls -la /etc/fail2ban/filter.d/
   ```

3. **AWX job failures**
   - Verify SSH connectivity
   - Check sudo permissions
   - Review inventory variables
   - Validate playbook syntax

### Support Resources
- GitHub Issues: Report bugs and feature requests
- Documentation: Comprehensive guides and examples
- Community: Join discussions and get help

## Next Steps

After successful deployment:
1. Monitor fail2ban logs for activity
2. Adjust jail settings based on your environment
3. Set up log rotation and monitoring
4. Create backup and recovery procedures
5. Document your specific configuration

## Maintenance

### Regular Tasks
- Review banned IP lists
- Update jail configurations
- Monitor log file sizes
- Test backup/restore procedures
- Update the collection regularly

### Updates
- Monitor GitHub releases
- Test updates in staging first
- Review changelog for breaking changes
- Update AWX project after collection updates