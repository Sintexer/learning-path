
> [!info]
> Info stack is a bucket of notes I want to organize, but don't have time to. It is an informational dump.

## Explain each

Saga state table
Scheduled polling
Worker leases
Claim counters
Retry timestamps
Timeout handling
Recovery after crashes

- Java simple cache impl
- Java 21, 
- PostgreSQL (via jOOQ), 
- Redis, 
- GCP, 
- Event-Driven Architecture, 
- Flyway.

## Ansible

**Ansible** is an automation and configuration-management tool. It is commonly used to configure servers, deploy applications, install packages, and run operational tasks.

### Key characteristics

- Uses human-readable **YAML playbooks**
- Usually connects through **SSH** for Linux or **WinRM** for Windows
- Generally **agentless**—no Ansible agent is required on target machines
- Designed for repeatable and mostly idempotent operations
- Can manage many machines simultaneously

### Example playbook

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

Running the playbook ensures PostgreSQL is installed and running on the target servers.

### Common use cases

- Installing and updating software
- Managing configuration files
- Creating users and permissions
- Deploying applications
- Starting and stopping services
- Applying security patches
- Coordinating multi-server operations

---

## Terraform

**Terraform** is an **Infrastructure as Code (IaC)** tool used primarily to create and manage infrastructure through provider APIs.

It can manage resources in:

- AWS, Azure, and Google Cloud
- Kubernetes
- VMware
- GitHub
- Datadog
- PostgreSQL and many other systems

### Key characteristics

- Uses the declarative **HashiCorp Configuration Language**, or HCL
- Describes the desired infrastructure state
- Builds a dependency graph to determine resource order
- Maintains a **state file** mapping configuration to real infrastructure
- Uses providers to communicate with external APIs
- Shows planned changes before applying them

### Example

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "database_server" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"

  tags = {
    Name = "postgres-server"
  }
}
```

Typical commands:

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

- `init`: Downloads providers and initializes the project.
- `plan`: Previews proposed changes.
- `apply`: Creates or modifies infrastructure.
- `destroy`: Removes managed infrastructure.

---

## Ansible vs Terraform

| Topic | Terraform | Ansible |
|---|---|---|
| Primary purpose | Provision infrastructure | Configure systems and deploy software |
| Typical question | “What resources should exist?” | “How should these machines be configured?” |
| Language | HCL | YAML |
| State | Maintains a state file | Normally has no central state file |
| Execution | Uses provider APIs | Usually uses SSH or WinRM |
| Agent required | No agent on resources | Usually agentless |
| Change preview | Strong `terraform plan` workflow | Check mode is available, but less comprehensive |
| Typical resources | Networks, VMs, databases, load balancers | Packages, files, users, services, applications |

### Simplified distinction

- **Terraform creates the server.**
- **Ansible configures software inside the server.**

There is some overlap: Ansible can provision cloud resources, and Terraform provisioners can execute configuration commands. However, it is usually better to use each tool for its main strength.

---

## Using them together

A common workflow is:

1. Terraform creates:
   - Network and subnets
   - Virtual machines
   - Security groups
   - Load balancer
   - Managed PostgreSQL instance

2. Ansible configures:
   - OS packages
   - PostgreSQL settings
   - Monitoring agents
   - Application files
   - System services

```text
Terraform → Infrastructure
Ansible   → Configuration and deployment
```

In cloud-native environments, application images may instead be created using tools such as Packer and deployed through Kubernetes or a CI/CD system.

---

## Important interview concepts

### Idempotency

An operation is idempotent when running it repeatedly produces the same desired result without unnecessary changes.

For example:

```yaml
state: present
```

means “ensure this package exists,” rather than “install it again every time.”

Both tools aim for idempotent behavior, although scripts or imperative commands used inside them may not be idempotent.

### Terraform state

Terraform stores information about managed resources in a state file. For team environments, state is normally kept in a remote backend with access control and locking.

The state may contain sensitive information, so it must not be casually committed to Git.

### Infrastructure drift

Drift occurs when real infrastructure differs from the declared Terraform configuration—for example, when someone manually modifies a security group in the cloud console.

`terraform plan` compares the configuration, state, and current infrastructure to identify required changes.

### Ansible inventory

An inventory defines the hosts Ansible manages:

```ini
[database_servers]
db1.example.com
db2.example.com
```

Inventories can be static files or dynamically generated from cloud platforms.
