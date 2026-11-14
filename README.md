# Ansible KVM Host Automation

## Overview

This project automates the provisioning and configuration of a Debian 12 virtualization host using Ansible.

It prepares a bare Debian server for running virtual machines using KVM and libvirt.

### Features

- Initial host bootstrap and user management
- SSH configuration and hardening
- Secure credential management with Ansible Vault
- Installation of KVM and libvirt
- Firewall configuration using nftables
- Kernel parameter configuration
- Time synchronization using Chrony
- System logging configuration
- Default libvirt NAT network configuration
- Post-deployment validation

The playbook is designed to be idempotent, ensuring that repeated executions do not introduce unnecessary changes.

## Repository Structure

```text
kvm-host-automation/
├── ansible.cfg
├── site.yml
├── README.md
├── inventory/
│   ├── hosts.yml
│   └── group_vars/
│       ├── kvm_hosts.yml
│       └── kvm_hosts/
│           └── vault.yml
└── roles/
    └── kvm-host/
        ├── defaults/
        ├── tasks/
        ├── templates/
        ├── handlers/
        ├── meta/
        └── molecule/
```

## Prerequisites

**Ansible Controller:**
- Ansible installed
- SSH connectivity to the target host
- `sshpass` installed for SSH password authentication

Install the required packages on Debian-based controllers:

```bash
sudo apt update
sudo apt install -y ansible sshpass
```

**Target Host:**
- Debian 12 (Bookworm)
- x86_64 architecture
- Intel VT-x or AMD-V hardware virtualization support
- SSH server installed
- Initial administrative account named `user1`
- Access to the root account password for initial bootstrap

## Configuration

### 1. Inventory

Configure the target host in `inventory/hosts.yml`:

```yaml
all:
  children:
    kvm_hosts:
      hosts:
        kvm01:
          ansible_host: 192.168.188.144
          ansible_user: user1
          ansible_password: "{{ kvm_user1_password }}"
          ansible_python_interpreter: /usr/bin/python3
```

Replace the IP address with the actual target host address.

### 2. Ansible Vault

Sensitive credentials are stored in an encrypted Ansible Vault file.

Create the Vault file:

```bash
mkdir -p inventory/group_vars/kvm_hosts
ansible-vault create inventory/group_vars/kvm_hosts/vault.yml
```

Add the following variables:

```yaml
---
kvm_root_become_password: "ROOT_PASSWORD"
kvm_user1_password: "USER1_PASSWORD"
```

- `kvm_root_become_password`: Root password used during initial bootstrap with `su`.
- `kvm_user1_password`: Password of `user1`, used for both SSH authentication and sudo.

To modify the encrypted credentials later:

```bash
ansible-vault edit inventory/group_vars/kvm_hosts/vault.yml
```

### 3. Role Configuration

Default configuration variables are defined in:

```text
roles/kvm-host/defaults/main.yml
```

These include the hostname-related settings, timezone, administrative user, SSH settings, firewall management network, logging, kernel parameters, and virtualization options.

Review the configuration before deployment, especially the SSH management network allowed by the firewall.

## Deployment

Run the following commands from the project root directory.

**Check playbook syntax:**

```bash
ansible-playbook site.yml --syntax-check --ask-vault-pass
```

**Test connectivity:**

```bash
ansible kvm_hosts -m ping --ask-vault-pass
```

**Run the provisioning playbook:**

```bash
ansible-playbook site.yml --ask-vault-pass
```

Enter the Vault password when prompted. SSH and sudo credentials are retrieved automatically from Vault.

The playbook first bootstraps the target by installing sudo and adding `user1` to the sudo group. It then executes the `kvm-host` role to configure the virtualization host.