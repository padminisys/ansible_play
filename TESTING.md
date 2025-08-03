# Testing Guide for Fail2Ban Ansible Collection

This guide provides comprehensive instructions for testing the fail2ban Ansible collection using Molecule.

## Prerequisites

### System Requirements
- Docker installed and running
- Python 3.9+ installed
- Git installed

### Install Dependencies

```bash
# Install Python dependencies
pip install -r requirements-dev.txt

# Or install individually
pip install ansible-core molecule[docker] molecule-plugins[docker]
pip install docker requests ansible-lint yamllint
```

### Docker Setup

Ensure Docker is running:
```bash
# Start Docker (Linux)
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (requires logout/login)
sudo usermod -aG docker $USER

# Test Docker connectivity
docker --version
docker info
```

## Molecule Testing Framework

### Available Test Scenarios

1. **Default Scenario** (`molecule/default/`)
   - Tests basic fail2ban installation on CentOS Stream 9 and 10
   - Standard configuration with moderate security settings
   - Suitable for development and staging environments

2. **Production Scenario** (`molecule/production/`)
   - Tests production-ready configurations
   - Stricter security settings and longer ban times
   - Tests different server roles (webservers, dbservers)
   - Environment-specific configurations

### Test Structure

```
molecule/
├── default/
│   ├── molecule.yml          # Test configuration
│   ├── converge.yml          # Installation playbook
│   ├── prepare.yml           # Environment preparation
│   └── verify.yml            # Verification tests
├── production/
│   ├── molecule.yml          # Production test config
│   ├── converge.yml          # Production installation
│   └── verify.yml            # Production verification
└── requirements.yml          # Galaxy dependencies
```

## Running Tests

### Quick Test Commands

```bash
# Test default scenario
molecule test

# Test specific scenario
molecule test -s default
molecule test -s production

# Test all scenarios
molecule test -s default && molecule test -s production
```

### Step-by-Step Testing

```bash
# 1. Create test environment
molecule create -s default

# 2. Prepare test environment
molecule prepare -s default

# 3. Run the playbook
molecule converge -s default

# 4. Test idempotency
molecule idempotence -s default

# 5. Run verification tests
molecule verify -s default

# 6. Clean up
molecule destroy -s default
```

### Development Workflow

```bash
# For iterative development
molecule create -s default
molecule converge -s default

# Make changes to playbook/roles
# Re-run converge to test changes
molecule converge -s default

# Run verification
molecule verify -s default

# Clean up when done
molecule destroy -s default
```

## Test Scenarios Explained

### Default Scenario Tests

**Platforms:**
- CentOS Stream 10 (latest development)
- CentOS Stream 9 (stable)

**Test Coverage:**
- ✅ Package installation (fail2ban, EPEL)
- ✅ Service management (start, enable, status)
- ✅ Configuration file generation
- ✅ SSH jail configuration and activation
- ✅ iptables integration
- ✅ Log file monitoring
- ✅ Client connectivity (fail2ban-client ping)
- ✅ Configuration syntax validation

**Variables Tested:**
```yaml
fail2ban_default_bantime: 3600      # 1 hour
fail2ban_default_maxretry: 5        # 5 attempts
fail2ban_loglevel: INFO             # Standard logging
```

### Production Scenario Tests

**Platforms:**
- CentOS Stream 10 (production web server)
- CentOS Stream 9 (production database server)

**Additional Test Coverage:**
- ✅ Stricter security settings validation
- ✅ Environment-specific configurations
- ✅ Web server jail configurations (webservers group)
- ✅ Database server hardening (dbservers group)
- ✅ Email notification configuration
- ✅ Production-grade ban times and retry limits

**Production Variables:**
```yaml
# Web servers
fail2ban_default_bantime: 7200      # 2 hours
fail2ban_default_maxretry: 3        # Stricter

# Database servers  
fail2ban_default_bantime: 14400     # 4 hours
fail2ban_default_maxretry: 2        # Very strict
```

## Verification Tests

### What Gets Tested

1. **Service Status**
   - fail2ban service is active and enabled
   - fail2ban socket exists and is accessible
   - fail2ban-client responds to ping

2. **Configuration Validation**
   - Configuration files exist with correct permissions
   - Configuration syntax is valid
   - Jails are loaded and active

3. **Security Integration**
   - iptables rules are created
   - Log files are monitored
   - Ban/unban functionality works

4. **Environment-Specific Tests**
   - Production ban times are enforced
   - Group-specific jails are loaded
   - Security settings match environment requirements

### Sample Verification Output

```
TASK [Verify fail2ban service is running] ************************************
ok: [centos-stream-10] => {
    "msg": "✅ fail2ban service is active"
}

TASK [Test fail2ban client connectivity] *************************************
ok: [centos-stream-10] => {
    "msg": "✅ fail2ban client responds: pong"
}

TASK [Verify SSH jail is active] *********************************************
ok: [centos-stream-10] => {
    "msg": "✅ SSH jail is active and monitoring"
}
```

## Troubleshooting

### Common Issues

1. **Docker Permission Errors**
   ```bash
   # Add user to docker group
   sudo usermod -aG docker $USER
   # Logout and login again
   ```

2. **Container Creation Fails**
   ```bash
   # Check Docker is running
   sudo systemctl status docker
   
   # Check available images
   docker images
   
   # Pull required images manually
   docker pull quay.io/centos/centos:stream10-development
   docker pull quay.io/centos/centos:stream9
   ```

3. **Molecule Command Not Found**
   ```bash
   # Install molecule
   pip install molecule[docker] molecule-plugins[docker]
   
   # Check installation
   molecule --version
   ```

4. **Test Failures**
   ```bash
   # Run with verbose output
   molecule test -s default -- -vvv
   
   # Check logs
   molecule list -s default
   
   # Debug specific step
   molecule converge -s default -- -vvv
   ```

5. **Template Path Issues**
   ```bash
   # Ensure you're running from project root
   pwd  # Should show the collection root directory
   
   # Check template files exist
   ls -la templates/
   ```

### Debug Commands

```bash
# List molecule instances
molecule list

# Check molecule configuration
molecule check -s default

# Login to test container
molecule login -h centos-stream-10 -s default

# View logs
docker logs <container_id>

# Check container status
docker ps -a
```

## CI/CD Integration

### GitHub Actions

The collection includes automated testing via GitHub Actions:

```yaml
# .github/workflows/ci.yml
- Matrix testing across Python versions and Ansible versions
- Both default and production scenarios
- Artifact collection for test results
- Integration with security scanning
```

### Local CI Simulation

```bash
# Simulate CI pipeline locally
yamllint .
ansible-lint
molecule test -s default
molecule test -s production
```

## Performance Testing

### Test Execution Times

- **Default scenario**: ~5-8 minutes
- **Production scenario**: ~6-10 minutes
- **Full test suite**: ~15-20 minutes

### Optimization Tips

```bash
# Use specific scenarios for faster iteration
molecule test -s default

# Skip destroy for debugging
molecule test -s default --destroy=never

# Parallel testing (if resources allow)
molecule test -s default &
molecule test -s production &
wait
```

## Best Practices

### Test Development

1. **Start Simple**: Begin with default scenario
2. **Incremental Testing**: Test changes frequently
3. **Use Verify Tasks**: Add comprehensive verification
4. **Document Assumptions**: Comment complex test logic
5. **Clean Up**: Always destroy test environments

### Continuous Integration

1. **Matrix Testing**: Test multiple OS/Ansible versions
2. **Artifact Collection**: Save logs and results
3. **Fail Fast**: Stop on first failure in CI
4. **Security Scanning**: Include vulnerability checks
5. **Documentation**: Keep testing docs updated

## Contributing Tests

### Adding New Test Scenarios

1. Create new scenario directory: `molecule/new-scenario/`
2. Copy and modify existing `molecule.yml`
3. Create scenario-specific playbooks
4. Add verification tests
5. Update CI configuration
6. Document the new scenario

### Test Guidelines

- All tests must be idempotent
- Use descriptive task names
- Include both positive and negative tests
- Test error conditions
- Verify security configurations
- Document expected outcomes

## Support

- **Issues**: Report test failures via GitHub Issues
- **Documentation**: Check README.md and AWX_SETUP_GUIDE.md
- **Community**: Join discussions for testing help
- **Security**: Report security test issues privately

## Appendix

### Molecule Configuration Reference

```yaml
# molecule.yml structure
dependency:          # Galaxy requirements
driver:             # Docker driver config
platforms:          # Test containers
provisioner:        # Ansible configuration
verifier:           # Test verification
scenario:           # Test sequence
```

### Useful Commands Reference

```bash
# Molecule commands
molecule --help
molecule list
molecule check
molecule create
molecule prepare
molecule converge
molecule idempotence
molecule side_effect
molecule verify
molecule cleanup
molecule destroy
molecule test

# Docker commands
docker ps
docker logs <container>
docker exec -it <container> /bin/bash
docker system prune

# Debugging
ansible-playbook --check
ansible-playbook --diff
ansible-playbook -vvv