# Octopus Project Documentation

## Overview
Octopus is an Ansible-based deployment solution for a multi-service voting application. The project automates the setup of a robust infrastructure across five distinct machines, using native system services (systemd) instead of containerization.

## Project Architecture
The application is composed of five interdependent services:

1.  **Poll**: A Python/Flask web interface that collects user votes and pushes them to a Redis queue.
2.  **Redis**: A message broker that buffers incoming votes.
3.  **Worker**: A Java service that consumes votes from Redis and persists them into a PostgreSQL database.
4.  **PostgreSQL**: A relational database that stores the final vote results.
5.  **Result**: A Node.js web interface that reads from PostgreSQL to display live results.

## Technical Principles

### Service Management
Every service is deployed as a **systemd** unit. This ensures:
- **Resilience**: Services restart automatically if they crash.
- **Persistence**: Services start automatically on system boot.
- **Standardization**: Services are managed using standard `systemctl` commands.

### Configuration
In alignment with **12-Factor App** principles, configuration (hosts, ports, credentials) is handled exclusively through environment variables, keeping the application logic separate from the environment state.

### Security
Sensitive information, such as database credentials, is protected using **Ansible Vault**. This ensures that no clear-text secrets are stored in the repository.

## Repository Structure
- `playbook.yml`: The main Ansible playbook orchestrating the deployment.
- `production`: The inventory file defining the target hosts.
- `roles/`: Individual Ansible roles for each service (Poll, Redis, Worker, etc.).
- `group_vars/all/`: Global variables and encrypted secrets.

## Deployment
To deploy the infrastructure, run the following command:
```bash
ansible-playbook -i production playbook.yml --vault-password-file .vault_pass
```

## Idempotence
The deployment is fully **idempotent**. Running the playbook multiple times against the same infrastructure will result in no changes if the system is already in the desired state, ensuring predictable and stable environments.
