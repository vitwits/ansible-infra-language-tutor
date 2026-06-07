# Ansible Infrastructure Setup

This project provides basic server provisioning using Ansible. It is split into two phases:

- Bootstrap phase (one-time setup)
- Main provisioning phase (repeatable configuration)

---

## Prerequisites

- SSH access to the target server (e.g. user: `space`)
- SSH key pair available locally (`~/.ssh/id_ed25519.pub`)
- Ansible installed on your local machine

Check Ansible installation:

```bash
ansible --version
```

---

## Project Structure

```text
infra-ansible/
├── inventory.ini
├── bootstrap.yml
├── site.yml
└── roles/
    ├── common/
    │   └── tasks/
    │       └── main.yml
    └── nexus/
    │   └── tasks/
    │       └── main.yml
    ├── docker/
    │   └── tasks/
    │       └── main.yml
    └── jenkins/
    │   └── tasks/
    │       └── main.yml
    └── nginx/
    │   └── tasks/
    │       └── main.yml
    └── ansible/
        └── tasks/
            └── main.yml
```

---

## 1. Bootstrap Phase (first run only)

This phase creates a dedicated automation user (`ansible`), configures SSH access, and grants passwordless sudo.

### Run bootstrap:

```bash
ansible-playbook -i inventory.ini bootstrap.yml --ask-become-pass
```

You will be prompted for the sudo password of the existing user (e.g. `space`).

---

## 2. Update Inventory

After bootstrap is complete, update `inventory.ini`:

```ini
[servers]
myserver ansible_host=YOUR_SERVER_IP ansible_user=ansible
```

From this point onward, Ansible will use the `ansible` user.

---

## 3. Main Provisioning Phase

This phase configures the server (updates system and installs base packages).

### Run provisioning:

```bash
ansible-playbook -i inventory.ini site.yml
```

No password will be required if SSH keys and sudo are configured correctly.

### Run a specific role only

If you want to run only a specific part of the configuration (e.g., only update Jenkins), use tags:

```bash
ansible-playbook -i inventory.ini site.yml --tags jenkins
```

---

## What this setup does

### Bootstrap phase:
- Creates `ansible` user
- Adds SSH public key access
- Configures passwordless sudo

### Main provisioning:
- Updates system packages
- Installs basic utilities:
  - git
  - curl
  - vim
  - unzip
  - nginx

### Docker role:
- Installs Docker Engine and Docker Compose plugin
- Ensures Docker service is started and enabled

### Jenkins role:
- Creates `/opt/jenkins` directory
- Deploys Jenkins via Docker Compose
- Starts the Jenkins container

---

## Notes

- This setup is idempotent (safe to re-run)
- Bootstrap is executed only once per server
- Main playbook can be run repeatedly
