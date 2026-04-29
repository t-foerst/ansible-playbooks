# Ansible Playbooks

Ansible playbooks for managing a homelab infrastructure built on Proxmox VE, running a K3s Kubernetes cluster with supporting services.

## Infrastructure Overview

```
Proxmox VE Cluster (14 nodes)
├── K3s Kubernetes Cluster
│   ├── Masters (2x Ubuntu)   10.10.20.200–201
│   ├── Workers (3x Ubuntu)   10.10.20.205–207
│   └── PostgreSQL Backend    10.10.20.211 (Debian)
├── Supporting Services
│   ├── MinIO Object Storage  10.10.20.213 (Debian)
│   ├── Private Proxy         10.10.20.212 (Debian)
│   └── Public Proxy          10.10.20.214 (Debian)
└── Exposed Nodes 
    ├── 10.10.40.110 (Debian)
    └── 10.10.40.120 (Debian)
```

## Prerequisites

- SSH access to all managed hosts
- Python 3 available on all managed hosts

## Configuration

The `ansible.cfg` points the inventory to `./inventory/hosts`. No further configuration is needed beyond ensuring SSH connectivity and appropriate sudo privileges on the target hosts.

## Inventory

The inventory is defined in `inventory/hosts` and contains the following groups:

| Group | Description | Hosts |
|-------|-------------|-------|
| `pvenodes` | All Proxmox VE nodes | 14 nodes |
| `debian` | Debian-based hosts | 6 hosts |
| `ubuntu` | Ubuntu-based hosts | 5 hosts |
| `masters` | K3s master nodes | 10.10.20.200–201 |
| `workers` | K3s worker nodes | 10.10.20.205–207 |
| `k3s` | All K3s nodes (masters + workers) | 5 hosts |
| `k3sdb` | PostgreSQL backend for K3s | 10.10.20.211 |
| `minio` | MinIO object storage | 10.10.20.213 |
| `proxys` | All proxy nodes | 10.10.20.212, 214 |
| `privateProxy` | Internal proxy | 10.10.20.212 |
| `publicProxy` | Internet-facing proxy | 10.10.20.214 |
| `exposed` | Internet-facing nodes | 10.10.40.110, 120 |

## Playbooks

### `playbooks/apt.yml` — System Updates

Updates APT cache and runs a full dist-upgrade on all Debian and Ubuntu hosts.

```bash
ansible-playbook playbooks/apt.yml
```

### `playbooks/update_exposed.yml` — Update Exposed Services

Same as `apt.yml` but scoped to the `exposed` group (internet-facing nodes). Run this more frequently to keep public-facing systems patched.

```bash
ansible-playbook playbooks/update_exposed.yml
```

### `playbooks/startup_cluster.yml` — Start K3s Cluster

Starts the K3s cluster in the correct order to avoid dependency failures:

1. Start PostgreSQL on `k3sdb`
2. Wait for port 5432 to be ready
3. Start K3s server on `masters`
4. Start K3s agent on `workers`

```bash
ansible-playbook playbooks/startup_cluster.yml
```

### `playbooks/shutdown_cluster.yml` — Shut Down K3s Cluster

Gracefully shuts down the K3s cluster in reverse order to prevent data corruption:

1. Stop K3s agent on `workers`
2. Stop K3s server on `masters`
3. Stop PostgreSQL on `k3sdb`

```bash
ansible-playbook playbooks/shutdown_cluster.yml
```
