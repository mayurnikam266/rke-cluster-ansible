# Automated RKE2 Kubernetes Cluster Deployment

Minimal, simple, and production-ready Ansible playbooks for deploying a High-Availability (HA) **RKE2 Kubernetes Cluster**.

---

## 🔄 Deployment Workflow

```text
[ inventory.ini ] ➔ Defines servers (master) & agents (worker)
        │
        ▼
[ Step 1: Base Setup ] ➔ Sets system hostnames automatically & installs packages (curl, jq)
        │
        ▼
[ Step 2: Master Setup ] ➔ Configures primary server (rke2-server-01) & joins additional servers
        │
        ▼
[ Step 3: Agent Setup ]  ➔ Configures & joins worker nodes (rke2-agent-01, 02) to the cluster
```

---

## ⚡ Step-by-Step Setup Guide

### 1. Configure Inventory (`inventory/inventory.ini`)
Define your control plane nodes under `[rke2_servers]` and worker nodes under `[rke2_agents]`:

```ini
[rke2_servers]
rke2-server-01 ansible_host=172.31.84.173 ansible_connection=local
rke2-server-02 ansible_host=172.31.93.75 ansible_user=ubuntu
rke2-server-03 ansible_host=172.31.85.255 ansible_user=ubuntu

[rke2_agents]
rke2-agent-01 ansible_host=172.31.80.131 ansible_user=ubuntu
rke2-agent-02 ansible_host=172.31.82.50 ansible_user=ubuntu
```

> 💡 **Automated Hostnames**: Ansible automatically sets each node's OS hostname from your inventory names (`rke2-server-01`, `rke2-agent-01`, etc.).

---

### 2. Configure Cluster Variables (`inventory/group_vars/all.yaml`)
Set your cluster token and Virtual IP / Load Balancer IP:

```yaml
rke2_token: "KtRTwtgcEP5hcZ8yoG3FQ4RlCjhC0I2dr+qqYqMEhAI="
rke2_vip: "172.31.93.207"
```

---

### 3. Deploy the Cluster (1 Command)
Run the master playbook from your project root:

```bash
ansible-playbook -i inventory/inventory.ini playbooks.yaml
```

---

## ➕ Adding a New Node Later

To add a new worker node (e.g. `rke2-agent-03`):

1. **Add IP to inventory** (`inventory/inventory.ini`):
   ```ini
   [rke2_agents]
   rke2-agent-01 ansible_host=172.31.80.131 ansible_user=ubuntu
   rke2-agent-02 ansible_host=172.31.82.50 ansible_user=ubuntu
   rke2-agent-03 ansible_host=172.31.88.99 ansible_user=ubuntu  # <-- New node
   ```

2. **Run setup targeting only the new node**:
   ```bash
   ansible-playbook -i inventory/inventory.ini playbooks.yaml --limit rke2-agent-03
   ```

---

## 🔍 Verification & Access

Log into `rke2-server-01` to view your running cluster nodes:

```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
/var/lib/rancher/rke2/bin/kubectl get nodes
```
