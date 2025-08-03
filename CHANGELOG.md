# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2025-08-03

### Added
- Initial release of the fail2ban Ansible collection
- Complete playbook for fail2ban installation and configuration on CentOS 10 Stream
- AWX/Ansible Tower integration support
- Comprehensive jail configurations for SSH, HTTP, and other services
- Environment-specific variable files (production, staging, development)
- Jinja2 templates for fail2ban.local and jail.local configuration
- Inventory examples with host groups and variables
- Extensive documentation with AWX setup guide
- CI/CD pipeline with GitHub Actions
- Security scanning and linting integration
- Molecule testing framework setup
- YAML and Ansible linting configuration

### Features
- **CentOS 10 Stream Support**: Optimized for latest CentOS Stream
- **Security Hardened**: Follows security best practices and CIS benchmarks
- **Idempotent Operations**: All tasks are fully idempotent and safe to re-run
- **AWX Ready**: Pre-configured for AWX/Ansible Tower deployment
- **Multi-Environment**: Separate configurations for dev/staging/production
- **Comprehensive Monitoring**: Built-in validation and status checking
- **Email Notifications**: Configurable email alerts for security events
- **Flexible Jails**: Easy-to-configure jail system for various services

### Security
- Encrypted credential management with AWX integration
- IP whitelisting support for management networks
- Secure default configurations with option to customize
- Audit logging and monitoring capabilities
- Integration with system firewall (iptables/firewalld)

### Documentation
- Complete AWX integration guide
- Step-by-step setup instructions
- Configuration variable reference
- Troubleshooting guide
- Security considerations and best practices
- Contributing guidelines

### Testing
- Molecule testing framework
- Multi-platform testing (CentOS, RHEL)
- CI/CD pipeline with automated testing
- Security vulnerability scanning
- YAML and Ansible linting

[Unreleased]: https://github.com/company/ansible-fail2ban-collection/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/company/ansible-fail2ban-collection/releases/tag/v1.0.0