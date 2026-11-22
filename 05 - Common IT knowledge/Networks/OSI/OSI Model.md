> AKA **Open Systems Interconnection**

The **Open Systems Interconnection (OSI)** model is a conceptual framework that describes the seven (7) layers of network communication. It was developed by the [[ISO]] in the 1980s to promote interoperability between different computer systems.

| Layer | Name | PDU | Examples | TCP/IP Model |
| --- | --- | --- | --- | --- |
| 7 | [[OSI 7 layer\|Application]] | Data | [[HTTP]], [[DNS]], [[gRPC]], [[WebSocket]] | Application |
| 6 | [[OSI 6 layer\|Presentation]] | Data | [[TLS]], [[Data Encoding]] (UTF-8, JSON, [[Protocol Buffers]]), gzip | Application |
| 5 | [[OSI 5 layer\|Session]] | Data | RPC sessions, TLS sessions, sockets | Application |
| 4 | [[OSI 4 layer\|Transport]] | Segment / Datagram | [[TCP]], [[UDP]], L4 [[Load Balancing]] | Transport |
| 3 | [[OSI 3 layer\|Network]] | Packet | IPv4/IPv6, ICMP, BGP, routers, [[Subnet]], [[CIDR Notation\|CIDR]] | Internet |
| 2 | [[OSI 2 layer\|Data Link]] | Frame | Ethernet, Wi-Fi, MAC, ARP, switches, [[VLAN]] | Link |
| 1 | [[OSI 1 layer\|Physical]] | Bits | Fiber, copper cables, radio waves, NICs | Link |

> [!tip] Production reality
> The real internet runs on the 4-layer **TCP/IP model** (where OSI layers 5–7 collapse into Application and layers 1–2 into Link). Still, engineers constantly use OSI layer numbers as shorthand—especially **L2** (switching/[[VLAN]]), **L3** (IP routing/[[Subnet]]), **L4** ([[TCP]]/[[UDP]] [[Load Balancing]]), and **L7** ([[HTTP]]/[[gRPC]] [[Reverse proxy]] / [[API Gateway]] / [[Service Mesh]]).

## Flashcards

OSI Model:::Open Systems Interconnection — 7-layer conceptual model of networking (1 Physical, 2 Data Link, 3 Network, 4 Transport, 5 Session, 6 Presentation, 7 Application).

What are the Protocol Data Units (PDUs) for OSI Layers 1 through 4?::L1: ==Bits==, L2: ==Frames==, L3: ==Packets==, L4: ==Segments (TCP) / Datagrams (UDP)==.

When debugging a network outage, work bottom-up or top-down across OSI layers (link -> IP/route -> TCP port -> TLS -> HTTP) to isolate the failing boundary fast. #card-reminder

## Related
- [[ISO]]
- [[TCP]]
- [[UDP]]
- [[Load Balancing]]
- [[TLS]]
