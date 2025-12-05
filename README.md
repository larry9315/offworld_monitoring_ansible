# Offworld Monitoring – Ansible Provisioning

This repository automates the provisioning of **monitoring servers** for Offworld using Ansible.  
The solution is fully **idempotent**, **repeatable**, and **production-aware**, capable of both initial provisioning and ongoing configuration management.

The playbook deploys:

- **Ubuntu Server 24.04.1 LTS** (minimal)
- A primary sysadmin account: **`offworld-admin`**
- A base set of administration and troubleshooting packages
- **Docker Engine**
- A full monitoring stack running as Docker containers:
  - **Prometheus**
  - **node-exporter**
  - **cAdvisor**
  - **Grafana** (with datasource + dashboard auto-provisioned)

---

1. Requirements

Control Node Requirements
• Ansible Core 2.17 or newer
• Python 3.x
• community.docker Ansible collection installed (run: ansible-galaxy collection install community.docker)

Target Node Requirements
• Fresh installation of Ubuntu Server 24.04.1 LTS (minimal)
• Must have the provisioning user:
username: ansible_provision
password: Granite1Pecan2Snore

This provisioning user must be in the sudo group. It is used exclusively by Ansible to configure the machine.

---

2. Inventory Configuration

Create the file: inventory/monitoring_servers.ini

Example contents:

[monitoring_servers]
monitor1 ansible_host=192.168.64.2

[monitoring_servers:vars]
ansible_user=ansible_provision
ansible_password=Granite1Pecan2Snore
ansible_become=yes
ansible_become_password=Granite1Pecan2Snore
ansible_connection=ssh

Modify the ansible_host value to point to your real machine.

This is the only required manual change before running the playbook.

---

3. Configurable Variables

All major variables reside in: group_vars/monitoring_servers.yml

Sysadmin Account Configuration
offworld_admin_username: offworld-admin
offworld_admin_password: Marble1Pine2Cough

SSH Authorized Keys
offworld_admin_authorized_keys:
• ssh-ed25519 AAAA…replace_this… user@machine

Replace the placeholder with real SSH keys before use.

System Administration Tools Installed
Examples include: curl, wget, vim, htop, git, tcpdump, net-tools, lsof, jq

Monitoring Stack Configuration
monitoring_base_dir: /opt/monitoring
monitoring_docker_network: monitoring_net

prometheus_port: 9090
node_exporter_port: 9100
cadvisor_port: 8080
grafana_port: 3000

Docker Image Versions
prometheus_image: prom/prometheus:latest
node_exporter_image: prom/node-exporter:latest
cadvisor_image: gcr.io/cadvisor/cadvisor:latest
grafana_image: grafana/grafana-oss:latest

---

4. Role Overview

Role: common
Installs administration and troubleshooting packages.

Role: users
Creates the offworld-admin account, installs authorized SSH keys, configures passwordless sudo, and prepares the user’s .ssh directory.

Role: docker
Installs Docker Engine and Python Docker SDK, ensures Docker is enabled and running, creates /opt/monitoring, and creates the dedicated Docker network monitoring_net. Also adds the offworld-admin user to the docker group.

Role: monitoring
Deploys Prometheus configuration, Grafana provisioning files (datasource and dashboard provider), dashboard JSON, and starts all required containers:
• node-exporter
• cAdvisor
• Prometheus
• Grafana
All containers run on the monitoring_net Docker network.

---

5. Running the Playbook

From the project root directory:

ansible-playbook site.yml

The playbook performs the following actions: 1. Updates apt cache 2. Installs common admin tools 3. Creates and configures the sysadmin account 4. Installs and configures Docker 5. Deploys the monitoring stack 6. Ensures all services and containers are running 7. Converges the system into its desired state

---

6. Verifying the Deployment

On the target machine, run:

docker ps
docker network ls | grep monitoring_net

Expected containers:

• grafana
• prometheus
• cadvisor
• node-exporter

Prometheus Web Interface
http://(VM IP):9090

Grafana Web Interface
http://(VM IP):3000

Grafana Login Credentials
username: admin
password: ChangeMeGrafana1

Grafana automatically loads a dashboard showing host and Docker metrics.
