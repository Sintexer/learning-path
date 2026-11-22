---
flashcards:
  q-mndn: { nid: 1795356723091, hash: tam5jc2v, sync: cr6axugx }
  q-zqxy: { nid: 1795361719280, hash: rhf6yv8f, sync: 2evwr45s }
  q-9sqj: { nid: 1795356723169, hash: zih4cyui, sync: p4c2brfd }
  q-wdyt: { nid: 1795356723194, hash: sys88jrg, sync: fibhnejf }
---

> AKA **Transmission Control Protocol**

**TCP** is a connection-oriented, reliable, ordered, byte-stream transport protocol at [[OSI 4 layer]]. It underpins [[HTTP]] (1.1 & 2), [[gRPC]], [[WebSocket]], SSH, and most [[Database]] wire protocols.

## Core Mechanisms

- **3-Way Handshake**: `SYN` -> `SYN-ACK` -> `ACK` (costs 1 RTT before application data flows; establishes initial sequence numbers and window sizes).
- **Reliability & Ordering**: every byte has a sequence number; receivers send cumulative `ACK`s (and SACKs); lost segments are retransmitted on timeout or 3 duplicate ACKs.
- **Flow Control (Receiver Window / `rwnd`)**: prevents a fast sender from overwhelming a slow receiver's kernel buffer (a form of transport-level [[Backpressure]]).
- **Congestion Control (`cwnd`)**: prevents overwhelming the network path via slow start, congestion avoidance, and congestion algorithms (CUBIC, BBR). Effective send window = \(\min(\text{cwnd}, \text{rwnd})\).
- **4-Way Teardown**: `FIN` / `ACK` in each direction; the active closer enters **`TIME_WAIT`** (typically 60s = 2 MSL) to ensure delayed packets from the old connection don't corrupt a new connection reusing the same 4-tuple.

## TCP vs [[UDP]]

- TCP guarantees delivery, ordering, and congestion control at the cost of handshake latency, kernel state, and **Head-of-Line (HoL) blocking** (one lost packet stalls delivery of all subsequent packets in the stream).
- [[UDP]] skips handshakes and reliability, enabling lower latency and custom transport protocols like QUIC ([[HTTP3]]).

> [!warning] Failure modes & trade-offs
> - **TCP Head-of-Line blocking**: even though [[HTTP2]] multiplexes independent streams over one TCP connection, a single lost TCP packet stalls *every* stream until retransmitted (solved by QUIC/[[UDP]] in [[HTTP3]]).
> - **Ephemeral port exhaustion & `TIME_WAIT`**: short-lived outbound connections (e.g., proxy to backend without keep-alive pooling) exhaust the ~28k–60k ephemeral port range on a 4-tuple. Reuse persistent connection pools.
> - **Nagle's algorithm + Delayed ACK**: combining Nagle (buffering small writes) with Delayed ACK (40–200ms) adds huge artificial latency to request-response RPCs; set `TCP_NODELAY` on latency-sensitive sockets.
> - **Silent dead peers (half-open connections)**: without TCP keepalives or application-level heartbeats/timeouts, a peer that crashes or drops behind a NAT can leave sockets stuck open indefinitely.

## Flashcards

TCP:::Transmission Control Protocol — reliable, ordered, connection-oriented byte-stream protocol at [[OSI 4 layer]] with flow control and congestion control.
^q-mndn

What is the difference between TCP flow control (`rwnd`) and congestion control (`cwnd`)?::Flow control protects the ==receiver== from buffer overflow (`rwnd`), while congestion control protects the ==network path== from overload (`cwnd`); the sender is limited by \(\min(\text{cwnd}, \text{rwnd})\).
^q-k32j
^q-zqxy

Why does the active closer of a TCP connection enter `TIME_WAIT`?::To absorb delayed in-flight packets from the closed connection so they are not misdelivered to a new connection reusing the same 4-tuple (src IP, src port, dst IP, dst port).
^q-9sqj

### Why can Nagle's algorithm combined with Delayed ACK cause 40–200ms latency spikes? #card

Nagle buffers small outbound writes until the previous packet is ACKed, while the receiver's Delayed ACK waits 40–200ms hoping to piggyback the ACK on a reply. Together they deadlock until the Delayed ACK timer fires. Fix by enabling `TCP_NODELAY` (and writing complete frames with writev/buffering).
^q-wdyt

## Related
- [[UDP]]
- [[OSI 4 layer]]
- [[TLS]]
- [[HTTP2]]
- [[HTTP3]]
- [[Backpressure]]
