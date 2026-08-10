# RKE2 Cluster Deployment with Ansible

Automated Ansible playbooks and roles for deploying a High-Availability (HA) RKE2 (Rancher Kubernetes Engine 2) cluster on Linux hosts.

---

## Directory Structure & Roles Explanation

```text
rke-cluster-ansible/
├── ansible.cfg
├── playbooks.yaml
├── inventory/
│   ├── inventory.ini
│   └── group_vars/
│       ├── all.yaml
│       ├── rke2_servers.yaml
│       └── rke2_agents.yaml
├── playbooks/
│   ├── 1Nodes_packages_setup.yaml
│   └── 2rke2_config_setup.yaml
└── roles/
    ├── rke2_master_config/
    ├── rke2_worker-config/
    ├── rke2-install-master/
    └── rke2-install-worker/
```

### Ansible Roles Overview

* **`roles/rke2_master_config`**: Creates `/etc/rancher/rke2/config.yaml` on master nodes. It configures the cluster token, sets the `server` endpoint for joiner masters, and registers all master IPs under `tls-san` to prevent SSL certificate errors.
* **`roles/rke2-install-master`**: Downloads the RKE2 installation script, runs it with `INSTALL_RKE2_TYPE=server`, and enables/starts `rke2-server.service`.
* **`roles/rke2_worker-config`**: Creates `/etc/rancher/rke2/config.yaml` on worker nodes, pointing them to the primary master or load balancer IP (`rke2_vip`) on port `9345`.
* **`roles/rke2-install-worker`**: Downloads the RKE2 installation script, runs it with `INSTALL_RKE2_TYPE=agent`, and enables/starts `rke2-agent.service`.

---

## Network & Port Requirements

Ensure security groups or firewalls allow inbound traffic between nodes on the following ports:

* **9345/TCP**: RKE2 Server Registration API (Required for joiner masters and worker nodes).
* **6443/TCP**: Kubernetes API Server.
* **10250/TCP**: Kubelet metrics / logs.
* **2379-2380/TCP**: etcd client and peer communication (Master nodes only).
* **8472/UDP**: Flannel VXLAN overlay network.

---

## Step-by-Step Deployment Guide

### Step 1: Define Inventory (`inventory/inventory.ini`)

Edit `inventory/inventory.ini` and specify your control plane servers under `[rke2_servers]` and worker agents under `[rke2_agents]`:

```ini
[rke2_servers]
rke2-server-01 ansible_host=172.31.84.173 ansible_connection=local
rke2-server-02 ansible_host=172.31.93.75 ansible_user=ubuntu
rke2-server-03 ansible_host=172.31.85.255 ansible_user=ubuntu

[rke2_agents]
rke2-agent-01 ansible_host=172.31.80.131 ansible_user=ubuntu
rke2-agent-02 ansible_host=172.31.82.50 ansible_user=ubuntu
```

### Step 2: Configure Variables (`inventory/group_vars/all.yaml`)

Edit `inventory/group_vars/all.yaml`:

```yaml
rke2_token: "KtRTwtgcEP5hcZ8yoG3FQ4RlCjhC0I2dr+qqYqMEhAI="
rke2_vip: "172.31.84.173"
```

> **IMPORTANT**: Set `rke2_vip` to the IP address of your primary master node (`rke2-server-01`, e.g., `172.31.84.173`) or your Load Balancer IP address. Secondary master nodes and worker nodes use this IP to join the cluster on port `9345`.

### Step 3: Run Deployment

Run the master playbook from your project root:

```bash
ansible-playbook -i inventory/inventory.ini playbooks.yaml
```

---

## Adding a New Node Later

### Adding a New Worker Agent (`rke2-agent-03`)

1. Append the new node to `inventory/inventory.ini`:
   ```ini
   [rke2_agents]
   rke2-agent-01 ansible_host=172.31.80.131 ansible_user=ubuntu
   rke2-agent-02 ansible_host=172.31.82.50 ansible_user=ubuntu
   rke2-agent-03 ansible_host=172.31.88.99 ansible_user=ubuntu
   ```

2. Execute the playbook targeting only the new node:
   ```bash
   ansible-playbook -i inventory/inventory.ini playbooks.yaml --limit rke2-agent-03
   ```

### Adding a New Server Node (`rke2-server-04`)

1. Append the new server to `[rke2_servers]` in `inventory/inventory.ini`.
2. Execute the playbook targeting only the new server node:
   ```bash
   ansible-playbook -i inventory/inventory.ini playbooks.yaml --limit rke2-server-04
   ```

---

## Troubleshooting & Verification

### Verify Cluster Status
Run on the primary master node:

```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
/var/lib/rancher/rke2/bin/kubectl get nodes
```

### Inspect RKE2 Logs
If a service fails to start:

* **Master Nodes**:
  ```bash
  journalctl -u rke2-server.service -f
  ```
* **Worker Nodes**:
  ```bash
  journalctl -u rke2-agent.service -f
  ```
