---
flashcards:
  q-ijv2: { nid: 1795356713768, hash: vvhsijzq, sync: 8xsnucvx }
  q-8z5p: { nid: 1795361650532, hash: wsjt49bz, sync: ntjddxvx }
  q-v75m: { nid: 1795361650603, hash: adqu4e62, sync: pb4jq46n }

---

**Ansible** is an open-source automation and configuration-management tool ([[IaaC]]). It is commonly used to configure servers, deploy applications, install packages, and orchestrate operational runbooks across fleets of machines.

## Key characteristics

- Uses human-readable **YAML playbooks**
- Connects through **SSH** (Linux) or **WinRM** (Windows) — **push-based** by default
- **Agentless** — no daemon or agent needs to be installed on target machines (only Python on Linux targets)
- Built around declarative-style modules (`apt`, `service`, `template`) that aim for [[Idempotency|idempotent]] execution
- Can manage many machines concurrently via forks/rolling batches (`serial`)

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

- Installing and updating OS packages and security patches
- Managing configuration files (Jinja2 templates)
- Creating users, SSH keys, and permissions
- Bootstrapping bare-metal or [[Virtual machine|VM]] hosts (e.g., [[Kubernetes]] nodes)
- Coordinating rolling multi-server operational tasks

## Ansible inventory

An inventory defines the hosts and groups Ansible manages:

```ini
[database_servers]
db1.example.com
db2.example.com
```

Inventories can be static files or dynamically generated from cloud provider APIs (AWS EC2, GCP).

## [[Idempotency]]

An operation is idempotent when running it repeatedly produces the same desired state without unnecessary changes. For example, `state: present` means "ensure this package exists" rather than "install it again every time." Imperative `shell` or `command` tasks inside playbooks are **not** automatically idempotent unless guarded with `creates:` / `changed_when:`.

> [!warning] Failure modes & trade-offs
> - **Mutable configuration drift**: because Ansible mutates live servers in place and maintains no central state file, removing a task from a playbook does *not* automatically undo what it previously installed on hosts (unlike [[Terraform]]).
> - **SSH scaling bottleneck**: push-based SSH execution slows down across thousands of nodes; use pipelining, Mitogen, or move to immutable images (Packer + [[Container|containers]]).
> - **Secrets in playbooks**: encrypt credentials with **Ansible Vault** or fetch them at runtime from an [[External Configuration Store]]—never commit plaintext secrets.

## Flashcards

Ansible:::Agentless, push-based configuration management and automation tool using YAML playbooks over SSH/WinRM.
^q-ijv2

Why does removing a task from an Ansible playbook *not* remove the resource from target servers (unlike [[Terraform]])?::Because Ansible has ==no central state file== tracking previously managed resources; you must explicitly write a task with `state: absent`.
^q-msjm
^q-8z5p

Why are Ansible `shell`/`command` tasks risky for repeatable automation?::They execute imperatively on every run and break ==[[Idempotency]]== unless you add explicit guards (`creates`, `removes`, `changed_when`) or use native Ansible modules.
^q-jypa
^q-v75m

## Related
- [[Terraform]]
- [[Ansible vs Terraform]]
- [[IaaC]]
- [[Idempotency]]
- [[Virtual machine]]
