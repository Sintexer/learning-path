**Ansible** is an automation and configuration-management tool. It is commonly used to configure servers, deploy applications, install packages, and run operational tasks.

## Key characteristics

- Uses human-readable **YAML playbooks**
- Usually connects through **SSH** for Linux or **WinRM** for Windows
- Generally **agentless**—no Ansible agent is required on target machines
- Designed for repeatable and mostly idempotent operations
- Can manage many machines simultaneously

## Example playbook

```yaml
- name: Configure PostgreSQL servers
  hosts: database_servers
  become: true

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.apt:
        name: postgresql
        state: present

    - name: Start PostgreSQL
      ansible.builtin.service:
        name: postgresql
        state: started
        enabled: true
```

Running the playbook ensures [[PostgreSQL]] is installed and running on the target servers.

## Common use cases

- Installing and updating software
- Managing configuration files
- Creating users and permissions
- Deploying applications
- Starting and stopping services
- Applying security patches
- Coordinating multi-server operations

## Ansible inventory

An inventory defines the hosts Ansible manages:

```ini
[database_servers]
db1.example.com
db2.example.com
```

Inventories can be static files or dynamically generated from cloud platforms.

## Idempotency

An operation is idempotent when running it repeatedly produces the same desired result without unnecessary changes. For example, `state: present` means “ensure this package exists,” rather than “install it again every time.” Scripts or imperative `shell`/`command` tasks inside playbooks are not automatically idempotent.

## Related

- [[Terraform]]
- [[Ansible vs Terraform]]
- [[IaaC]]
- [[Idempotency]]
