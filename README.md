Ansible Server Baseline
Automated server hardening and baseline configuration using Ansible.
A production-ready Ansible collection for automated server provisioning, security hardening, monitoring setup, and Docker installation.

📊 Architecture Overview
<img width="8818" height="2633" alt="Architecture-1" src="https://github.com/user-attachments/assets/629bec06-11cd-41f9-90e8-6b4b42e41422" />

    
🎯 Features
🔒 Security Hardening
    • SSH Security: Disable root login, enforce key-based authentication, configure strong ciphers
    • Firewall Configuration: Automatic UFW/Firewalld setup with customizable rules
    • User Management: Create sudo users, configure SSH keys, set password policies
    • System Hardening: Disable unused services, configure auditd, set kernel parameters
    • Security Updates: Automatic installation of security patches
📊 Monitoring & Observability
    • Node Exporter: Prometheus metrics exporter for system monitoring
    • Custom Metrics: Application-specific metric collection
    • Log Rotation: Automated log management with logrotate
    • Resource Monitoring: Disk, CPU, memory, and network monitoring
🐳 Containerization Ready
    • Docker Installation: Latest Docker CE and Docker Compose
    • User Configuration: Add users to docker group (optional)
    • Storage Setup: Configure Docker daemon options
    • Container Networking: Pre-configured bridge networks
⚙️ System Configuration
    • Time Synchronization: NTP/Chrony configuration
    • Locale Settings: Timezone and language configuration
    • Package Management: Apt/Yum/DNF repository configuration
    • Performance Tuning: Sysctl optimization for production
📁 Project Structure
text
ansible-server-baseline/
├── inventories/			# Environment definitions
│   ├── production.yml		# Production servers
│   ├── staging.yml			# Staging environment
│   └── development.yml		# Development servers
├── roles/				# Ansible roles
│   ├── hardening/			# Security hardening
│   │   ├── tasks/main.yml		# Main tasks
│   │   ├── templates/		# Configuration templates
│   │   │   ├── sshd_config.j2
│   │   │   └── audit.rules.j2
│   │   └── defaults/main.yml	# Role defaults
│   ├── monitoring/			# Monitoring setup
│   │   ├── tasks/main.yml
│   │   ├── templates/
│   │   │   └── node_exporter.service.j2
│   │   └── vars/
│   │       └── main.yml
│   ├── docker/			# Docker installation
│   │   ├── tasks/main.yml
│   │   └── defaults/main.yml
│   ├── updates/			# System updates
│   │   └── tasks/main.yml
│   └── common/			# Common configurations
│       ├── tasks/main.yml
│       └── defaults/main.yml
├── site.yml				# Main playbook
├── playbooks/			# Additional playbooks
│   ├── deploy.yml
│   └── validate.yml
├── group_vars/			# Group variables
│   ├── all.yml
│   └── webservers.yml
├── host_vars/			# Host-specific variables
│   └── example-server.yml
├── .github/workflows/		# CI/CD pipelines
│   └── ansible-test.yml
├── tests/				# Test suite
│   ├── test.yml
│   └── requirements.yml
├── docs/				# Documentation
│   ├── architecture.md
│   └── best-practices.md
├── .gitignore				# Git ignore rules
├── ansible.cfg			# Ansible configuration
├── requirements.yml			# Role dependencies
└── README.md			# This file
🚀 Quick Start
Prerequisites
    • Ansible 2.9+ installed on control node
    • Python 3.6+ on target servers
    • SSH access with sudo privileges to target servers
    • Inventory file with target servers defined
Installation
    1. Clone the repository:
       git clone https://github.com/strizhenko/ansible-server-baseline.git
       cd ansible-server-baseline
       
    2. Install role dependencies:
       ansible-galaxy install -r requirements.yml
    3. Create inventory file:
       cp inventories/production.example.yml inventories/production.yml
       
    4. Edit inventory file:
       # inventories/production.yml
       all:
         hosts:
           web-server-01:
             ansible_host: 192.168.1.100
             ansible_user: admin
           db-server-01:
             ansible_host: 192.168.1.101
             ansible_user: admin
         
         vars:
           # Global variables
           ssh_port: 2222
           admin_users:
             - name: deploy
               ssh_key: "ssh-rsa AAAAB3NzaC1yc2…"
       
    5. Run the playbook:
       # Dry run (check mode)
       ansible-playbook -i inventories/production.yml site.yml --check
       
       # Actual execution
       ansible-playbook -i inventories/production.yml site.yml
       
       # With tags (run specific roles)
       ansible-playbook -i inventories/production.yml site.yml --tags="hardening,monitoring"
Example: Single Server Deployment
For quick testing on a local VM or cloud instance:
# Create minimal inventory
cat > inventory.ini << EOF
[servers]
test-server ansible_host=10.0.0.1 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

[servers:vars]
ssh_port=22
enable_docker=true
enable_monitoring=true
EOF

# Run with tags
ansible-playbook -i inventory.ini site.yml --tags="common,hardening,docker"
⚙️ Configuration
Role-Specific Variables
Hardening Role
# group_vars/all.yml
hardening:
  ssh:
    port: 2222
    permit_root_login: "no"
    password_authentication: "no"
    allow_users: ["deploy", "admin"]
  
  firewall:
    enabled: true
    default_policy: "deny"
    allowed_ports:
      - port: "{{ ssh_port }}"
        proto: tcp
      - port: 80
        proto: tcp
      - port: 443
        proto: tcp
  
  users:
    - name: deploy
      groups: sudo
      ssh_keys:
        - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
      sudo_nopasswd: true
    
    - name: monitor
      groups: docker
      shell: /bin/bash
Monitoring Role
monitoring:
  node_exporter:
    enabled: true
    version: "1.6.0"
    port: 9100
    collectors:
      - cpu
      - memory
      - diskstats
      - filesystem
      - netstat
      - systemd
    
    extra_args:
      - "--collector.textfile.directory=/var/lib/node_exporter/textfile_collector"
      - "--web.listen-address=:9100"
  
  custom_metrics:
    enabled: true
    scripts:
      - name: docker_containers
        script: |
          #!/bin/bash
          echo '# HELP docker_containers_total Total Docker containers'
          echo '# TYPE docker_containers_total gauge'
          echo "docker_containers_total $(docker ps -q | wc -l)"
Docker Role
docker:
  version: "24.0"
  compose_version: "v2.20.0"
  
  users:
    - deploy
    - admin
  
  storage:
    driver: "overlay2"
    options:
      - "overlay2.override_kernel_check=true"
  
  daemon_config:
    log-driver: "json-file"
    log-opts:
      max-size: "10m"
      max-file: "3"
    storage-driver: "overlay2"
    live-restore: true
Updates Role
updates:
  automatic:
    enabled: true
    security_only: true
    reboot:
      enabled: true
      time: "03:00"
      require: "security"
  
  unattended_upgrades:
    enabled: true
    mail: "admin@example.com"
    remove_unused_deps: true
    auto_reboot: false
Customizing for Different OS
# group_vars/ubuntu.yml (for Ubuntu servers)
package_manager: apt
systemd: true
service_manager: systemctl

# group_vars/rhel.yml (for RHEL/CentOS servers)
package_manager: yum
systemd: true
service_manager: systemctl
firewall_service: firewalld
🎮 Usage Examples
1. Complete Server Provisioning
# Provision all roles on all servers
ansible-playbook -i inventories/production.yml site.yml
2. Security Hardening Only
# Apply only security hardening
ansible-playbook -i inventories/production.yml site.yml --tags="hardening"
3. Docker Setup on Specific Group
# Install Docker on webservers group
ansible-playbook -i inventories/production.yml site.yml \
  --limit webservers \
  --tags="docker"
4. Monitoring Setup with Custom Port
# Set up monitoring with custom Node Exporter port
ansible-playbook -i inventories/production.yml site.yml \
  --tags="monitoring" \
  --extra-vars="node_exporter_port=19100"
5. Dry Run with Verbose Output
# Test changes without applying
ansible-playbook -i inventories/production.yml site.yml \
  --check \
  --diff \
  -vvv
🔧 Advanced Usage
Using Vault for Secrets
# Create encrypted variables file
ansible-vault create group_vars/secrets.yml

# Edit encrypted file
ansible-vault edit group_vars/secrets.yml

# Run playbook with vault password
ansible-playbook -i inventories/production.yml site.yml \
  --ask-vault-pass
  
# Or use password file
ansible-playbook -i inventories/production.yml site.yml \
  --vault-password-file ~/.vault_pass
Dynamic Inventory
# Use AWS EC2 dynamic inventory
ansible-playbook -i aws_ec2.yml site.yml

# Use custom Python inventory script
ansible-playbook -i inventory.py site.yml
Role Testing with Molecule
# Install molecule
pip install molecule molecule-docker

# Test hardening role
cd roles/hardening
molecule test

# Create scenario
molecule init scenario -r hardening -d docker
📊 Verification
Check Applied Configuration
# Verify SSH configuration
ansible all -i inventories/production.yml -m shell \
  -a "sshd -T | grep -E 'permitrootlogin|passwordauthentication'"

# Check Docker installation
ansible all -i inventories/production.yml -m shell \
  -a "docker --version && docker-compose --version"

# Verify Node Exporter
ansible all -i inventories/production.yml -m uri \
  -a "url=http://localhost:9100/metrics return_content=yes"
Generate Report
# playbooks/report.yml
- name: Generate configuration report
  hosts: all
  tasks:
    - name: Gather facts
      setup:
    
    - name: Create report
      template:
        src: report.j2
        dest: "/tmp/system-report-{{ inventory_hostname }}.md"
      delegate_to: localhost
🧪 Testing
Local Testing with Vagrant
# Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.inventory_path = "inventories/vagrant.yml"
    ansible.extra_vars = {
      ssh_port: 2222,
      enable_docker: true
    }
  end
end
CI/CD Pipeline
# .github/workflows/ansible-test.yml
name: Ansible Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install Ansible
      run: pip install ansible ansible-lint yamllint
    
    - name: Lint playbooks
      run: |
        ansible-lint site.yml
        yamllint -c .yamllint .
    
    - name: Syntax check
      run: ansible-playbook site.yml --syntax-check
    
    - name: Test with Molecule
      run: |
        cd roles/hardening
        molecule test
📈 Monitoring Dashboard
After deploying the monitoring role, access Prometheus metrics:
    1. Node Exporter Metrics: http://your-server:9100/metrics
    2. Prometheus Configuration:
yaml
# prometheus.yml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['your-server:9100']
    3. Grafana Dashboard Import: Use dashboard ID 1860 (Node Exporter Full)
🛡️ Security Best Practices
SSH Key Management
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_deploy

# Add to authorized_keys with restrictions
echo 'restrict,command="/bin/false" ssh-ed25519 AAAAB3NzaC1yc2E...' >> ~/.ssh/authorized_keys
Firewall Rules
# Minimal required ports
hardening:
  firewall:
    allowed_ports:
      - { port: "{{ ssh_port }}", proto: tcp, comment: "SSH" }
      - { port: 9100, proto: tcp, comment: "Node Exporter", source: "monitoring_net" }
      - { port: 9090, proto: tcp, comment: "Prometheus", source: "monitoring_net" }
Audit Logging
hardening:
  audit:
    enabled: true
    rules:
      - "-w /etc/passwd -p wa -k identity"
      - "-w /etc/shadow -p wa -k identity"
      - "-w /etc/sudoers -p wa -k sudoers"
      - "-a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F auid!=4294967295 -k perm_mod"
🔄 Update Management
Update Process
# Check for available updates
ansible all -i inventories/production.yml -m apt \
  -a "update_cache=yes upgrade=safe" --become

# Apply security updates only
ansible-playbook -i inventories/production.yml playbooks/security-updates.yml

# Schedule automatic updates
ansible-playbook -i inventories/production.yml site.yml --tags="updates"
Rollback Procedure
# Create backup before changes
ansible-playbook -i inventories/production.yml playbooks/backup.yml

# Rollback SSH configuration
ansible all -i inventories/production.yml -m copy \
  -a "src=/etc/ssh/sshd_config.backup dest=/etc/ssh/sshd_config mode=0644" \
  --become
🤝 Contributing
We welcome contributions! Please see our Contributing Guide.
Development Setup
# Fork and clone
git clone https://github.com/your-username/ansible-server-baseline.git
cd ansible-server-baseline

# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements-dev.txt

# Create feature branch
git checkout -b feature/awesome-feature

# Run tests
make test
Pull Request Process
    1. Update README.md with details of changes if needed
    2. Add tests for new functionality
    3. Ensure all tests pass
    4. Update version in meta/main.yml
    5. Create Pull Request with description
📚 Documentation
    • Architecture Details
    • Role Reference
    • Best Practices
    • Troubleshooting
    • FAQs
🐛 Troubleshooting
Common Issues
SSH Connection Problems:
# Test SSH connection
ansible all -i inventory.ini -m ping

# Debug SSH
ansible all -i inventory.ini -m raw -a "whoami" -vvv
Permission Denied:
# Use become with password
ansible-playbook -i inventory.ini site.yml --become --ask-become-pass

# Check sudo configuration
ansible all -i inventory.ini -m shell -a "sudo -l"
Package Installation Failures:
# Retry with different package manager
package_manager_force: true
use_epel: true  # For RHEL/CentOS
Debugging
# Increase verbosity
ansible-playbook -i inventory.ini site.yml -vvv

# Step-by-step execution
ansible-playbook -i inventory.ini site.yml --step

# Start at specific task
ansible-playbook -i inventory.ini site.yml --start-at-task="Install Docker"
📄 License
This project is licensed under the MIT License - see the LICENSE file for details.
👤 Author
Oleksandr Stryzhenko - Infrastructure/Cloud Engineer
    • GitHub: @strizhenko
    • LinkedIn: oleksandr-stryzhenko
    • Email: strizhenkoalexander@gmail.com
🙏 Acknowledgments
    • Inspired by CIS Benchmarks
    • Based on Ansible Best Practices
    • Uses community Ansible roles from Ansible Galaxy
Version: 1.0.0
Ansible Version: 2.9+
Supported OS: Ubuntu 18.04+, Debian 10+, RHEL/CentOS 7+

