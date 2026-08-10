# RKE2 Cluster Deployment - Step-by-Step Guide

Automated Ansible deployment for RKE2 Kubernetes High-Availability Cluster.

---

##  Prerequisites
- Ansible installed on the control machine.
- SSH access to target nodes (`ubuntu` user with sudo access or `root`).

---

##  Inventory Setup (`inventory/inventory.ini`)

Configure your control plane (`rke2_servers`) and worker nodes (`rke2_agents`) with explicit hostnames, IP addresses (`ansible_host`), connection modes, and users:

```ini
[rke2_servers]
rke2-server-01 ansible_host=172.31.84.173 ansible_connection=local
rke2-server-02 ansible_host=172.31.93.75 ansible_user=ubuntu
rke2-server-03 ansible_host=172.31.85.255 ansible_user=ubuntu

[rke2_agents]
rke2-agent-01 ansible_host=172.31.80.131 ansible_user=ubuntu
rke2-agent-02 ansible_host=172.31.82.50 ansible_user=ubuntu
```

---

##  Run Deployment

Execute the master playbook:

```bash
ansible-playbook -i inventory/inventory.ini playbooks.yaml
```

---

## ➕ Adding a New Agent Node Later

1. Open `inventory/inventory.ini` and append the new agent under `[rke2_agents]`:
   ```ini
   [rke2_agents]
   rke2-agent-01 ansible_host=172.31.80.131 ansible_user=ubuntu
   rke2-agent-02 ansible_host=172.31.82.50 ansible_user=ubuntu
   rke2-agent-03 ansible_host=172.31.88.99 ansible_user=ubuntu  # <-- New worker node
   ```

2. Run the deployment targeting only the new agent:
   ```bash
   ansible-playbook -i inventory/inventory.ini playbooks.yaml --limit rke2-agent-03
   ```
