# High Availability PostgreSQL Database with Ansible, Patroni, Etcd, and HAProxy

This repository contains an **Ansible playbook collection** for deploying a highly available PostgreSQL cluster using [Patroni](https://patroni.readthedocs.io/), [Etcd](https://etcd.io/), and [HAProxy](http://www.haproxy.org/).  
It automates the setup of PostgreSQL 16, Patroni configuration, cluster coordination with Etcd, and HAProxy load balancing for both read/write and read-only queries.

---

## 🚀 How to create credentials

```bash
ansible-vault create vars/pg_creds.yml
```

set your password, then copy paste some string like this :

```
pg_credentials:
  replication:
    username: replicator
    password: "ChangeMeOrDie"
  superuser:
    username: postgres
    password: "ChangeMeOrDie"
  admin:
    username: admin
    password: "ChangeMeOrDie"
```

to edit :

```bash
 ansible-vault edit vars/pg_creds.yml
```

## 🚀 How to run

```bash
ansible-playbook -u ubuntu -i inventory.ini install-cluster.yaml
```

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
├── ansible.cfg
├── install-cluster.yml
├── inventory.ini
├── playbook.yml
├── README.md
├── templates/
│ ├── etcd.j2
│ ├── haproxy-lb-yml.j2
│ ├── haproxy.conf.j2
│ └── patroni.yml.j2
└── vars/
└── pg_vars.yml

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

## 🏗️ Architecture

```
                 ┌─────────────┐
                 │   Clients   │
                 └──────┬──────┘
                        │
                ┌───────┴────────┐
                │    HAProxy      │
                │ (RW 5432 / RO 5433)
                └───┬─────────┬──┘
                    │         │
     ┌──────────────┘         └──────────────┐
     │                                      │
┌────▼─────┐                         ┌──────▼─────┐
│ pg-node-1│                         │ pg-node-2  │
│ Patroni  │                         │ Patroni    │
│ Postgres │                         │ Postgres   │
└────┬─────┘                         └──────┬─────┘
     │ Leader Election via Etcd            │
     └───────────────┬─────────────────────┘
                     │
               ┌─────▼─────┐
               │ pg-node-3 │
               │ Patroni   │
               │ Postgres  │
               └───────────┘

```

## 📝 Notes

Designed for Ubuntu 22.04+ / 24.04 hosts.

Tested with PostgreSQL 16.

Etcd and Patroni communicate over the private network.

Use firewall rules to secure external access.

## 🤝 Contributing

Feel free to fork this project and submit PRs for improvements (e.g. support for Consul/Zookeeper as DCS, TLS for HAProxy, or Docker/Kubernetes deployments).

## 📜 License

MIT License – you are free to use, modify, and distribute this project.
