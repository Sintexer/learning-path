> AKA **Virtual Local Area Network**

A **VLAN** (standardized in **IEEE 802.1Q**) logically partitions a single physical [[LAN]] switch infrastructure into multiple isolated Layer 2 ([[OSI 2 layer]]) broadcast domains.

## How It Works

- **802.1Q Tagging**: switches insert a 4-byte tag containing a 12-bit **VLAN ID** (up to 4,094 usable VLANs) into Ethernet frames traveling across **trunk ports** (switch-to-switch or switch-to-[[Hypervisor]] links).
- **Access ports**: end-host ports belong to a single untagged VLAN; the switch strips/adds the tag transparently.
- **Inter-VLAN routing**: devices in different VLANs cannot talk directly at L2 even if plugged into the same physical switch—traffic must pass through a Layer 3 router or L3 switch (where ACLs/firewalls can filter it).

## VLAN vs [[Subnet]] vs [[Virtual Private Cloud|VPC]]

- **VLAN** = Layer 2 broadcast-domain isolation (physical/on-prem switches).
- **[[Subnet]]** = Layer 3 IP address range (often mapped 1:1 with a VLAN on-prem, or software-defined inside a [[Virtual Private Cloud|VPC]]).
- **VXLAN / Overlay networks** = encapsulate L2 frames inside [[UDP]] packets to break past the 4,094 VLAN limit across multi-tenant cloud fabrics and [[Kubernetes]] clusters.

> [!warning] Failure modes & trade-offs
> - **4,094 VLAN ceiling**: traditional 802.1Q runs out in large multi-tenant data centers, which is why clouds use VXLAN/Geneve overlays.
> - **VLAN hopping & misconfigured trunks**: leaving unused switch ports in trunk mode or on default VLAN 1 allows attackers to inject tagged frames across segments.

## Flashcards

VLAN:::Virtual Local Area Network (IEEE 802.1Q) — logical segmentation of a physical [[LAN]] into separate Layer 2 broadcast domains using a 12-bit VLAN ID tag.

Devices in different VLANs on the same physical switch must communicate through a ==Layer 3 router== (enabling firewalling and isolation between segments).

What is the maximum number of VLANs supported by IEEE 802.1Q, and how do cloud providers exceed it?::4,094 (12-bit VLAN ID); cloud providers use overlay encapsulation protocols like ==VXLAN== or Geneve over L3/[[UDP]].

## Related
- [[LAN]]
- [[Subnet]]
- [[OSI 2 layer]]
- [[Virtual Private Cloud]]
