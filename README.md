# proxmox-monitoring-stack

Ansible-based deployment for Prometheus, Alertmanager, Grafana, pve-exporter, node exporter targets, and Ceph monitoring.

Run this **after** completing [proxmox-prepare](https://github.com/45Drives/proxmox-prepare) on your Proxmox hosts.
To be done in the VM
---

## Prerequisites

When booting the VM copy the ssh key from the VM to all of the Proxmox nodes

```bash
ssh-keygen
ssh-copy-id root@<PVE IP>
```

### Proxmox nodes
Ensure you have already run the `proxmox-prepare` playbook on your Proxmox hosts and created a Prometheus API token:

```bash
pveum user add prometheus@pve --comment "Prometheus monitoring user"
pveum user token add prometheus@pve monitoring --privsep 0
pveum aclmod / --users prometheus@pve --roles PVEAuditor
```

Save the token value from the output — you will need it below.

### Monitoring VM

Create a VM to run the monitoring stack (Ubuntu 22 recommended), then install the dependencies:

```bash
apt update
apt install -y git ansible jq curl
```

---

## Installation

### 1. Clone the repo

```bash
cd /opt
git clone https://github.com/45Drives/proxmox-monitoring-stack.git
cd /opt/proxmox-monitoring-stack
```

### 2. Set up config files

```bash
cp inventories/local/group_vars/all/vault.yml.example inventories/local/group_vars/all/vault.yml
cp inventories/local/hosts.yml.example inventories/local/hosts.yml
```

### 3. Edit the config files

Add the Prometheus API token value you saved earlier:

```bash
vim inventories/local/group_vars/all/vault.yml
```

```yaml
vault_pve_token_value: "5f9f61a5-874f-4bbb-8eb7-07d926bee8idn"
```

Add your email settings:
```bash
vim inventories/local/group_vars/all/main.yml
```

### 4. Edit the inventory

Replace the hostnames and IPs with your own Proxmox environment:

```bash
vim inventories/local/hosts.yml
```

### 5. Run the playbook

```bash
cd /opt/proxmox-monitoring-stack
ansible-playbook site.yml
```

---

## Verify

Once the playbook completes, confirm the stack is running:

```bash
docker ps
```

All monitoring containers should be up and running.

> _**NOTE:_** the config files can all be found in the monitoring root (ex. /opt/monitoring/prometheus/prom.yml)
