---
flashcards:
  q-sgj3: { nid: 1795361647061, hash: brn82jgf, sync: 8y7zfve2 }
  q-vp9t: { nid: 1795361647137, hash: awe8vr48, sync: wr3gqvan }

---

| Topic | [[Terraform]] | [[Ansible]] |
| --- | --- | --- |
| Primary purpose | Provision cloud/platform infrastructure | Configure OS/software and run operational tasks |
| Typical question | "What resources should exist?" | "How should these machines be configured?" |
| Paradigm | Declarative, immutable-leaning | Procedural/hybrid, mutable-in-place |
| Language | HCL | YAML |
| State | Maintains a central state file (with locking) | Normally stateless (no central state file) |
| Execution | Calls provider REST/gRPC APIs | Usually push-based over SSH or WinRM |
| Agent required | No agent on resources | Agentless (requires Python on Linux targets) |
| Change preview | Strong `terraform plan` graph diff | `--check` mode exists, but less reliable for shell tasks |
| Typical resources | [[Virtual Private Cloud\|VPCs]], [[Subnet\|subnets]], [[Virtual machine\|VMs]], [[Database\|databases]], [[Load Balancing\|load balancers]] | Packages, config files, users, systemd services, patches |

## Simplified distinction

- **[[Terraform]] creates the infrastructure.**
- **[[Ansible]] configures software inside the machines.**

There is some overlap: Ansible has cloud modules to provision resources, and Terraform has `provisioner` blocks to run scripts. However, HashiCorp explicitly recommends treating Terraform provisioners as a last resort—use each tool for its core strength.

## Using them together

A common workflow is:

1. [[Terraform]] creates:
   - [[Virtual Private Cloud|VPC]] and [[Subnet|subnets]]
   - [[Virtual machine|Virtual machines]]
   - Security groups & IAM roles
   - [[Load Balancing|Load balancer]]
   - Managed [[PostgreSQL]] instance

2. [[Ansible]] configures:
   - OS packages and kernel tuning
   - [[PostgreSQL]] settings
   - Monitoring / [[OpenTelemetry]] agents
   - Application config files and systemd services

```text
Terraform → Infrastructure provisioning (stateful graph)
Ansible   → Host configuration and orchestration
```

In [[Cloud-Native App|cloud-native]] environments, runtime host mutation via Ansible is often replaced by **immutable images** baked in CI (Packer for VM AMIs, OCI images for [[Container|containers]]) and deployed declaratively through [[Kubernetes]] + GitOps.

## Flashcards

How do [[Terraform]] and [[Ansible]] complement each other in an [[IaaC]] pipeline?::[[Terraform]] declaratively ==provisions infrastructure== (VPCs, VMs, databases, LBs) using a state file and provider APIs, while [[Ansible]] ==configures OS/software== inside hosts over SSH.
^q-bmys
^q-sgj3

In modern [[Cloud-Native App|cloud-native]] stacks, what largely replaces [[Ansible]] for mutating live servers?::==Immutable images== (Packer AMIs / [[Container|OCI containers]]) deployed declaratively via [[Kubernetes]] or autoscaling groups.
^q-44wx
^q-vp9t

## Related
- [[Terraform]]
- [[Ansible]]
- [[IaaC]]
- [[Cloud-Native App]]
- [[Kubernetes]]
