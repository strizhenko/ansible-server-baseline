# Best Practices Guide

## 1. Inventory Management

### Structure
- Keep production, staging, and development inventories separate
- Use group_vars and host_vars for variable organization
- Store sensitive data in Ansible Vault

### Example inventory structure:
inventories/
├── production/
│ ├── hosts.yml
│ ├── group_vars/
│ └── host_vars/
├── staging/
└── development/

text

## 2. Variable Management

### Variable Precedence
1. Command line values (`-e`)
2. Role defaults (`defaults/main.yml`)
3. Inventory variables
4. Playbook variables
5. Role variables (`vars/main.yml`)
6. Block variables
7. Task variables

### Naming Conventions
- Use snake_case for variable names
- Prefix role-specific variables with role name: `docker_version`, `monitoring_enabled`
- Use descriptive names: `webserver_max_connections`, `database_backup_retention_days`

## 3. Role Design

### Single Responsibility Principle
Each role should have one specific purpose:
- `hardening` - Security configuration
- `monitoring` - Monitoring setup
- `docker` - Container runtime
- `common` - Base system configuration

### Role Structure
role_name/
├── defaults/ # Default variables (lowest priority)
├── vars/ # Role variables
├── tasks/ # Main tasks
├── handlers/ # Handlers
├── templates/ # Jinja2 templates
├── files/ # Static files
└── meta/ # Dependencies

4. Playbook Design

### Tags
Use tags to control execution:
roles:
  - { role: hardening, tags: ['security', 'hardening'] }
  - { role: monitoring, tags: ['monitoring'] }

Error Handling

tasks:
  - name: Try something
    command: /bin/false
    ignore_errors: yes
    register: result
    
  - name: Check result
    debug:
      msg: "Command failed but we continue"
    when: result is failed

5. Security Best Practices
SSH Hardening
Disable root login

Use key-based authentication

Change default SSH port

Configure strong ciphers and MACs

Firewall Configuration
Default deny policy

Allow only necessary ports

Log all denied connections

Regular rule reviews

6. Performance Optimization
Pipelining
Enable in ansible.cfg:

[ssh_connection]
pipelining = True
Fact Caching

[defaults]
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts

7. Testing
Linting
ansible-lint site.yml
yamllint .

Syntax Check
ansible-playbook site.yml --syntax-check

Idempotency Test
ansible-playbook site.yml
ansible-playbook site.yml  # Should show 0 changes

8. Documentation
README Contents
Project description
Quick start guide
Configuration examples
Troubleshooting
Contributing guidelines

Inline Documentation

- name: Configure service
  template:
    src: service.conf.j2
    dest: /etc/service.conf
  # Purpose: Main service configuration
  # Variables: service_port, service_timeout
  # Dependencies: common role

9. Version Control
Git Workflow
Use feature branches

Write descriptive commit messages

Squash commits before merging

Tag releases

.gitignore
Exclude secrets and sensitive data

Ignore temporary files

Exclude IDE configurations

10. CI/CD Integration
GitHub Actions
Lint on every push

Syntax check on PR

Integration tests

Security scanning

Automated Testing
yaml
- name: Test with Molecule
  run: molecule test
