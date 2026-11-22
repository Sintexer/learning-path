> AKA **Classless Inter-Domain Routing**

`192.168.1.30` is a single IP address. If you want to express IP addresses between the range of `192.168.1.0` and `192.168.1.255`, how can you do that?

One way is to use CIDR notation. CIDR notation is a compressed way of representing a range of IP addresses (an IP prefix). Specifying a range determines how many IP addresses are available to you. It begins with a starting IP address and is separated by a forward slash (`/`) followed by a prefix length. The number at the end specifies how many of the leading bits of the IP address are fixed. In `192.168.1.0/24`, the first 24 bits are fixed and the last 8 bits are flexible (\(2^{32-24} = 256\) addresses).

**Example:** `192.168.1.0/24`

The higher the number after the `/`, the smaller the number of IP addresses in your network. For example, `192.168.1.0/24` (256 IPs) is much smaller than `192.168.1.0/16` (65,536 IPs).

## Quick IPv4 CIDR Reference

| Prefix | Total IPs | Usable (Standard) | Usable (AWS [[Virtual Private Cloud|VPC]]) | Common Use |
| --- | --- | --- | --- | --- |
| `/32` | 1 | 1 | 1 | Single host route / firewall rule |
| `/28` | 16 | 14 | 11 | Small service/transit [[Subnet]] |
| `/24` | 256 | 254 | 251 | Typical [[Subnet]] |
| `/16` | 65,536 | 65,534 | — | Entire [[Virtual Private Cloud|VPC]] |
| `/0` | All IPv4 | — | — | Default route (`0.0.0.0/0`) |

> [!warning] Failure modes & trade-offs
> - **Cloud reserved IPs**: standard networking reserves 2 IPs per [[Subnet]] (network + broadcast), but cloud providers like AWS reserve **5 IPs** per subnet (network, VPC router, [[DNS]], future use, broadcast).
> - **Overlapping CIDRs**: choosing the same RFC 1918 block (e.g., `10.0.0.0/16`) across multiple [[Virtual Private Cloud|VPCs]] or on-prem networks blocks VPC peering and [[VPN]] routing later.
> - **IP exhaustion in [[Kubernetes]]**: CNI plugins that assign VPC IPs directly to [[Pod|Pods]] (e.g., AWS VPC CNI) can exhaust a `/24` subnet quickly during rolling deployments.

## Flashcards

CIDR:::Classless Inter-Domain Routing — a compact notation (`IP/prefix`) specifying how many leading bits of an IP address are fixed as the network prefix.

How many total IPv4 addresses are in a `/N` CIDR block?::\(2^{32 - N}\) (e.g., `/24` = 256, `/16` = 65,536, `/32` = 1).

In AWS [[Virtual Private Cloud|VPC]] subnets, ==5== IP addresses are reserved per subnet (compared to ==2== in standard networking).

Plan non-overlapping CIDR ranges across VPCs and on-prem networks from day one so future VPC peering and VPNs work without NAT hacks. #card-reminder

## Related
- [[Subnet]]
- [[Virtual Private Cloud]]
- [[OSI 3 layer]]
- [[VPN]]
- [[Kubernetes]]
