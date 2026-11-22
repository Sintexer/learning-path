---
flashcards:
  q-3jwu: { nid: 1795356730224, hash: 5cn8u7yc, sync: 3z3c3dh4 }
  q-hken: { nid: 1795356730244, hash: xhzg8enf, sync: 884zivvv }
  q-a8a8: { nid: 1795356730266, hash: ntmptbgu, sync: aw6kej7v }
---

> AKA **Infrastructure as a Service**

**Infrastructure as a Service (IaaS)** is a cloud computing model that provides on-demand, pay-as-you-go virtualized compute, storage, and networking primitives (e.g., AWS EC2, EBS, [[Virtual Private Cloud|VPC]], GCP Compute Engine, Azure VMs).

## Shared Responsibility & Cloud Abstraction Stack

| Model | Cloud Provider Manages | You Manage | Example |
| --- | --- | --- | --- |
| **On-Premises** | Nothing | Hardware, network, [[Hypervisor]], OS, runtime, app | Bare-metal data center |
| **IaaS** | Physical hardware, data center, virtualization/[[Hypervisor]] | Guest OS, patching, runtime, middleware, app, data | AWS EC2, GCP Compute Engine |
| **PaaS** | Up through OS, runtime, and scaling platform | App code and data | Heroku, Elastic Beanstalk |
| **[[Serverless|FaaS / Serverless]]** | Everything up to function/container execution | Function/app code and event wiring | AWS Lambda, AWS Fargate |
| **SaaS** | Entire stack | Configuration and user data | Gmail, Salesforce |

> [!tip] Production reality
> IaaS gives maximum control (custom kernels, specialized networking, bare-metal/GPU tuning, predictable sustained-load pricing) at the cost of owning OS patching, security hardening, and scaling automation. Most teams provision IaaS declaratively via [[IaaC]] ([[Terraform]]) and layer [[Kubernetes]] or managed services on top to avoid maintaining raw [[Virtual machine|VMs]] by hand.

## Flashcards

IaaS (Infrastructure as a Service):::Cloud model providing virtualized compute, storage, and networking where the provider manages physical hardware and virtualization while the user manages the OS, runtime, and applications.
^q-3jwu

In the IaaS shared responsibility model, the cloud provider manages the ==physical hardware and hypervisor==, while the customer is responsible for ==guest OS patching, runtime, and application security==.
^q-hken

## When should you choose IaaS over PaaS or Serverless? #card

When you need OS/kernel-level control, custom networking, specialized hardware (GPUs, NVMe), predictable cost under steady high utilization, or to run workloads incompatible with serverless execution limits (long-running connections, large memory/state).
^q-a8a8

## Related
- [[Serverless]]
- [[Virtual machine]]
- [[Hypervisor]]
- [[IaaC]]
- [[Virtual Private Cloud]]
- [[Cloud-Native App]]
