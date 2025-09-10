# High Availability PostgreSQL Database with Ansible, Patroni, Etcd, and HAProxy

This repository contains an **Ansible playbook collection** for deploying a highly available PostgreSQL cluster using [Patroni](https://patroni.readthedocs.io/), [Etcd](https://etcd.io/), and [HAProxy](http://www.haproxy.org/).  
It automates the setup of PostgreSQL 16, Patroni configuration, cluster coordination with Etcd, and HAProxy load balancing for both read/write and read-only queries.

---

## ✨ Features

- Automated installation of **PostgreSQL 16**.
- **Patroni** for PostgreSQL high availability & automatic failover.
- **Etcd** as the Distributed Configuration Store (DCS).
- **HAProxy** as load balancer:
  - Port `5432` → **Leader (write)** connections.
  - Port `5433` → **Replicas (read-only)** connections.
- Automatic configuration of:
  - `postgresql.conf` (`listen_addresses = '*'`)
  - `pg_hba.conf` with replication and client access.
  - Dynamic `/etc/hosts` across all nodes.
- Secure credentials stored in `vars/pg_credentials.yml`.

---

## 📦 Project Structure

```
.
├── inventory.ini
├── playbooks/
│ ├── install-etcd.yml
│ ├── install-pg-patroni.yml
│ ├── deploy-haproxy.yml
│ └── set-hostnames.yml
├── templates/
│ ├── etcd.j2
│ ├── patroni.yml.j2
│ └── haproxy.conf.j2
└── vars/
├── pg_version.yml
└── pg_credentials.yml
```

---

## 🖥️ Inventory Example (`inventory.ini`)

```ini
[haproxy]
pg-haproxy ansible_host=10.11.12.105

[etcd]
pg-etcd ansible_host=10.11.12.104

[nodes]
pg-node-1 ansible_host=10.11.12.101
pg-node-2 ansible_host=10.11.12.102
pg-node-3 ansible_host=10.11.12.103
```

.
┌─────────────┐
│ Clients │
└──────┬──────┘
│
┌───────┴────────┐
│ HAProxy │
│ (RW 5432 / RO 5433)
└───┬─────────┬──┘
│ │
┌──────────────┘ └──────────────┐
│ │

┌────▼─────┐ ┌──────▼─────┐
│ pg-node-1│ │ pg-node-2 │
│ Patroni │ │ Patroni │
│ Postgres │ │ Postgres │
└────┬─────┘ └──────┬─────┘
│ Leader Election via Etcd │
└───────────────┬─────────────────────┘
│
┌─────▼─────┐
│ pg-node-3 │
│ Patroni │
│ Postgres │
└───────────┘
