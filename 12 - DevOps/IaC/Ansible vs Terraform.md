| Topic | [[Terraform]] | [[Ansible]] |
|---|---|---|
| Primary purpose | Provision infrastructure | Configure systems and deploy software |
| Typical question | “What resources should exist?” | “How should these machines be configured?” |
| Language | HCL | YAML |
| State | Maintains a state file | Normally has no central state file |
| Execution | Uses provider APIs | Usually uses SSH or WinRM |
| Agent required | No agent on resources | Usually agentless |
| Change preview | Strong `terraform plan` workflow | Check mode is available, but less comprehensive |
| Typical resources | Networks, VMs, databases, load balancers | Packages, files, users, services, applications |

## Simplified distinction

- **[[Terraform]] creates the server.**
- **[[Ansible]] configures software inside the server.**

There is some overlap: Ansible can provision cloud resources, and Terraform provisioners can execute configuration commands. However, it is usually better to use each tool for its main strength.

## Using them together

A common workflow is:

1. [[Terraform]] creates:
   - Network and subnets
   - [[Virtual machine|Virtual machines]]
   - Security groups
   - [[Load Balancing|Load balancer]]
   - Managed [[PostgreSQL]] instance

2. [[Ansible]] configures:
   - OS packages
   - PostgreSQL settings
   - Monitoring agents
   - Application files
   - System services

```text
Terraform → Infrastructure
Ansible   → Configuration and deployment
```

In [[Cloud-Native App|cloud-native]] environments, application images may instead be created using tools such as Packer and deployed through [[Kubernetes]] or a CI/CD system.
