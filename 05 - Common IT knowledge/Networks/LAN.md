> AKA **Local Area Network**

A **Local Area Network (LAN)** is a network that interconnects devices within a limited physical area (an office, home, or single data center rack/hall), typically over Ethernet (IEEE 802.3) or Wi-Fi (IEEE 802.11).

## Key Characteristics

- **Operates at [[OSI 1 layer|L1]] & [[OSI 2 layer|L2]]**: devices on the same LAN communicate directly using MAC addresses via Ethernet switches (no L3 router hop needed within the same broadcast domain).
- **Broadcast domain**: protocols like ARP (mapping IP to MAC) and DHCP rely on Layer 2 broadcasts across the LAN.
- **High bandwidth, low latency**: sub-millisecond RTT and multi-Gbps throughput compared to WAN links.
- **Segmentation**: large physical LANs are partitioned logically into [[VLAN|VLANs]] (L2) and [[Subnet|subnets]] (L3) to limit broadcast storms and enforce security boundaries.

> [!warning] Failure modes & trade-offs
> - **Broadcast storms & L2 loops**: a switching loop without Spanning Tree Protocol (STP) floods the entire LAN and brings down every connected host.
> - **Flat-network blast radius**: putting all servers in one flat LAN allows lateral movement via ARP spoofing or unrestricted port access; segment with [[VLAN|VLANs]], [[Subnet|subnets]], and [[Zero-Trust Security]].

## Flashcards

LAN:::Local Area Network — a high-speed, low-latency network connecting devices in a limited physical area within a shared Layer 2 broadcast domain.

Within the same LAN, hosts deliver frames directly using ==MAC addresses== (resolved from IPs via ==ARP==) without needing a Layer 3 router.

Why do large networks split a single physical LAN into [[VLAN|VLANs]] and [[Subnet|subnets]]?::To contain Layer 2 broadcast domains (preventing broadcast storms) and isolate security/traffic zones.

## Related
- [[VLAN]]
- [[Subnet]]
- [[OSI 2 layer]]
- [[Virtual Private Cloud]]
- [[VPN]]
