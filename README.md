# RKE2 Cluster Deployment - Step-by-Step Guide

Simple, automated Ansible deployment for RKE2 Kubernetes cluster.

---

##  Step 0: Prerequisites
- Ansible installed on your machine.
- Passwordless SSH access (`ssh-copy-id`) to target nodes as `root` (or sudo user).

---

##  Step 1: Edit Inventory (IPs Only)

Open `inventory/inventory.ini` and **just list your IP addresses**:

```ini
[rke2_master]
192.168.1.10

[rke2_agents]
192.168.1.20
192.168.1.21
```

>  **Automatic Hostnames**: You don't need to name them! Ansible automatically names them:
> - `192.168.1.10` ➔ `master-1`
> - `192.168.1.20` ➔ `worker-1`
> - `192.168.1.21` ➔ `worker-2`

---

##  Step 2: Run Full Setup in 1 Command

Execute the master playbook:

```bash
ansible-playbook -i inventory/inventory.ini playbooks.yaml
```

---

##  Adding a New Node Later

1. Add the new IP to `inventory/inventory.ini` under `[rke2_agents]`:
   ```ini
   [rke2_agents]
   192.168.1.20
   192.168.1.21
   192.168.1.22  # New node (auto-named worker-3)
   ```

2. Run setup targeting only the new node:
   ```bash
   ansible-playbook -i inventory/inventory.ini playbooks.yaml --limit 192.168.1.22
   ```
# rke-cluster-ansible
