> AKA **VPC**

A **Virtual Private Cloud (VPC)** is a logically isolated, software-defined virtual network provisioned within a public cloud provider (e.g., AWS VPC, GCP VPC, Azure VNet). It gives you full control over IP addressing ([[CIDR Notation]]), [[Subnet|subnets]], route tables, internet/NAT gateways, and firewall rules.

## Core Building Blocks

- **CIDR Block**: private IPv4/IPv6 address space (e.g., `10.0.0.0/16`) subdivided into public, private, and isolated [[Subnet|subnets]] across Availability Zones.
- **Route Tables**: determine next hops (`local` for intra-VPC traffic, Internet Gateway for public egress, NAT Gateway for private egress, Transit Gateway / Peering / [[VPN]] for cross-VPC or on-prem traffic).
- **Security Groups vs NACLs** (AWS model):
  - **Security Groups**: **stateful** instance/ENI-level firewalls (allow-only rules; return traffic is automatically permitted).
  - **Network ACLs (NACLs)**: **stateless** [[Subnet]]-level firewalls (evaluated in numbered order, support explicit deny, require explicit return-traffic rules).
- **VPC Endpoints (PrivateLink)**: reach cloud services (S3, DynamoDB, ECR) privately without traversing the public internet or paying NAT Gateway data-processing fees.

> [!warning] Failure modes & trade-offs
> - **NAT Gateway egress surprise bills**: routing heavy S3/ECR/container-image pulls from private subnets through a NAT Gateway incurs steep per-GB processing charges—add VPC Gateway/Interface Endpoints.
> - **Non-transitive VPC peering**: if VPC A peers with B, and B peers with C, A *cannot* route to C through B. Use a Transit Gateway (hub-and-spoke) when connecting many VPCs.
> - **Overlapping CIDRs**: deploying multiple VPCs with identical `10.0.0.0/16` ranges blocks peering and hybrid [[VPN]] connectivity later.

## Flashcards

VPC (Virtual Private Cloud):::A logically isolated virtual network in a public cloud with user-defined [[CIDR Notation|CIDR]] ranges, [[Subnet|subnets]], route tables, and security controls.

What is the difference between AWS Security Groups and Network ACLs?::Security Groups are ==stateful== allow-only firewalls at the instance/ENI level; NACLs are ==stateless== allow/deny firewalls at the [[Subnet]] level.

Why should private subnets use VPC Endpoints for services like S3 or container registries?::To keep traffic off the public internet and avoid high ==NAT Gateway== per-GB data-processing costs.

VPC peering is non-transitive; use a hub-and-spoke Transit Gateway when connecting more than a handful of VPCs. #card-reminder

## Related
- [[Subnet]]
- [[CIDR Notation]]
- [[VPN]]
- [[IaaS]]
- [[IaaC]]
- [[Zero-Trust Security]]
