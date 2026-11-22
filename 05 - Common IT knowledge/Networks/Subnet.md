---
flashcards:
  q-rbpm: { nid: 1795356723218, hash: 32fw5mbc, sync: nyaf6riw }
  q-cqyi: { nid: 1795356723241, hash: 556vag74, sync: qxikcund }
  q-pw9m: { nid: 1795356723267, hash: 7t9yatvc, sync: 2dhx8ny4 }
---

Imagine a **subnet** as a *smaller*, *isolated* network **within** your larger [[Virtual Private Cloud|VPC]] that inherits its overall network context. **Subnets** offer a way to **segment** your resources and **control** network **traffic flow**, enhancing both *security* and *efficiency*. Usually a subnet is a smaller [[LAN]] inside your base network, or paired with a [[VLAN]] in a traditional on-premises network. In an on-premises network, the typical use case for subnets is to isolate or optimize network traffic.

A subnet's [[CIDR Notation|CIDR]] IP range must be a subset of the parent network CIDR range. For example, if the parent network has CIDR range `192.168.1.0/24` and a subnet uses the same CIDR range, then only one subnet fits. By increasing the number of fixed bits in the CIDR range for a subnet, we leave alternative fixed bits for other subnets. With subnet CIDR range `192.168.1.128/25` we can have two subnets of equal size (`192.168.1.0/25` and `192.168.1.128/25`), or even more subnets with fewer flexible bits. That is the core principle of dividing a parent network into subnets.

**Why Use Subnets?**

* **Granular Control:** Subnets allow you to define specific route tables and network access rules (NACLs) for different groups of resources. For example, you can create a public subnet for applications that need internet access, a private subnet for resources that don't require external connectivity, and a database subnet for highly sensitive data.
* **Enhanced Security:** By isolating resources in different subnets, you can limit the potential impact of security breaches. If one subnet is compromised, the damage is contained within that subnet, preventing attackers from accessing other resources.
* **Improved Performance & Fault Isolation:** Subnets reduce broadcast traffic and, in cloud providers (like AWS), each subnet is scoped to a single Availability Zone (AZ), enabling multi-AZ high [[Availability]].

**Real-World Applications:**

* **Public Subnet:** Route table points `0.0.0.0/0` to an Internet Gateway (IGW). Hosts [[Load Balancing|load balancers]], NAT gateways, and bastion hosts.
* **Private Subnet:** Route table points `0.0.0.0/0` to a NAT Gateway (outbound-only internet access). Hosts application servers and [[Kubernetes]] worker nodes.
* **Isolated / Database Subnet:** No route to the internet at all. Hosts highly sensitive [[Database|databases]] accessible only from the application tier.

> [!warning] Failure modes & trade-offs
> - **Single-AZ blast radius**: in AWS, a subnet cannot span Availability Zones—deploy across at least 2–3 subnets in separate AZs for high availability.
> - **Undersized subnets**: you cannot resize an existing cloud subnet's CIDR in place; running out of IPs (common with [[Kubernetes]] pod networking) forces creating new subnets and migrating workloads.

## Flashcards

What distinguishes a public subnet from a private subnet in a cloud [[Virtual Private Cloud|VPC]]?::A public subnet has a route to an Internet Gateway (`0.0.0.0/0 -> IGW`) and public IPs; a private subnet routes outbound traffic through a NAT Gateway and cannot be reached directly from the internet.
^q-rbpm

Splitting a `/24` network into two equal subnets produces two ==`/25`== subnets (128 total IPs each).
^q-cqyi

Always size subnets with headroom for rolling deployments and [[Kubernetes]] pods, because cloud subnet CIDRs cannot be resized in place. #card-reminder
^q-pw9m

## Related
- [[CIDR Notation]]
- [[Virtual Private Cloud]]
- [[VLAN]]
- [[LAN]]
- [[OSI 3 layer]]
