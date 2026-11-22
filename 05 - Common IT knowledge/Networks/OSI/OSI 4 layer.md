---
flashcards:
  q-vry3: { nid: 1795356723675, hash: mpsum6b6, sync: t6hk2j8u }
  q-k7eg: { nid: 1795356723694, hash: p2tjtchy, sync: uvk98uv6 }
---

> AKA **Transport layer**

This [[OSI Model]] layer provides end-to-end communication between **processes/ports** on different hosts. Depending on the protocol, it offers segmentation, port multiplexing, flow control, congestion control, and reliable ordered delivery.

- **PDU**: Segment ([[TCP]]) / Datagram ([[UDP]])
- **Examples**: [[TCP]], [[UDP]], QUIC, L4 [[Load Balancing]] (NLB, `kube-proxy`).

## Flashcards

OSI Layer 4:::Transport layer — provides process-to-process communication via ports, with either reliable ordered streams ([[TCP]]) or lightweight datagrams ([[UDP]]).
^q-vry3

L4 [[Load Balancing]] routes connections based on ==IP addresses and TCP/UDP ports== without inspecting application payloads.
^q-k7eg

## Related
- [[OSI Model]]
- [[OSI 3 layer]]
- [[OSI 5 layer]]
- [[TCP]]
- [[UDP]]
- [[Load Balancing]]
