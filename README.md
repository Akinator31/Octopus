# G-DOP-400 — Octopus

Ansible-based deployment of the poll application across five machines, without containers.

## Overview

The poll application is composed of five services deployed on five distinct machines:

- **Poll** — Python/Flask web client that collects votes and pushes them to Redis
- **Redis** — message queue that buffers incoming votes
- **Worker** — Java service that consumes votes from Redis and persists them to PostgreSQL
- **PostgreSQL** — relational database storing the vote results
- **Result** — Node.js web client that reads from PostgreSQL and displays live results

Each service runs as a systemd unit and starts automatically on boot. Configuration is passed exclusively through environment variables (host, port, credentials, database name) in accordance with 12-factor app principles.

## Requirements

- Ansible installed on the control machine
- Five machines running Debian 10 (Buster)
- SSH access to all target machines
- Ansible Vault password available at `/tmp/.vault_pass`

Docker and Ansible Galaxy are not used in this project.

## Repository Structure

```
.
|-- production          # inventory file
|-- playbook.yml
|-- poll.tar
|-- result.tar
|-- worker.tar
|-- group_vars
|   `-- all.yml         # shared variables (vault-encrypted secrets)
`-- roles
    |-- base
    |   `-- tasks/main.yml
    |-- postgresql
    |   |-- files
    |   |   |-- pg_hba.conf
    |   |   `-- schema.sql
    |   `-- tasks/main.yml
    |-- redis
    |   |-- files
    |   |   `-- redis.conf
    |   `-- tasks/main.yml
    |-- poll
    |   |-- files
    |   |   `-- poll.service
    |   `-- tasks/main.yml
    |-- result
    |   |-- files
    |   |   `-- result.service
    |   `-- tasks/main.yml
    `-- worker
        |-- files
        |   `-- worker.service
        `-- tasks/main.yml
```

## Roles

### base

Installs essential system packages and configures the base instance. Applied to all hosts.

### redis

Installs Redis and deploys a custom `redis.conf`. The service is managed by systemd and bound only to the address reachable by the poll service.

### postgresql

Installs PostgreSQL 12 and the `psql` client. Creates the `paul` database user with a vaulted password and limited permissions, then initializes the schema from `schema.sql`.

### poll

Uploads the poll application archive, installs Python dependencies, and starts the Flask web client as a systemd service.

### worker

Uploads the worker archive, installs Java dependencies, builds the worker JAR, and runs it as a systemd service.

### result

Uploads the result application archive, installs Node.js dependencies, and starts the web client as a systemd service.

## Usage

Set the vault password, then run the playbook against the production inventory:

```bash
export ANSIBLE_VAULT_PASSWORD_FILE=/tmp/.vault_pass
echo verySecretPassword > /tmp/.vault_pass
ansible-playbook -i production playbook.yml
```

## Idempotence

Running the playbook a second time against an already-configured infrastructure must produce zero `changed` tasks in the play recap. All tasks are written to be idempotent.

## Security

Secrets (database passwords, etc.) are managed with Ansible Vault. No clear-text credentials are present anywhere in the repository. Any clear-text password found in the repository is grounds for immediate project failure during evaluation.
