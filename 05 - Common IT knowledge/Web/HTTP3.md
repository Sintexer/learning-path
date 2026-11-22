**Introduction:**
HTTP/3 (RFC 9114) is the latest major version of [[HTTP]], built on top of the **QUIC** transport protocol over [[UDP]] instead of [[TCP]]. It directly eliminates [[TCP]]-level Head-of-Line blocking, speeds up connection establishment, and handles mobile network transitions gracefully.

## Key Features & Improvements Over [[HTTP2]]

1. **QUIC over [[UDP]]**: Implements reliability, congestion control, multiplexing, and [[TLS]] 1.3 encryption in user space on top of [[UDP]].
2. **No Transport-Level Head-of-Line Blocking**: Each QUIC stream is independent—a lost packet only stalls the specific stream it belongs to, not other concurrent streams on the same connection.
3. **Faster Connection Establishment**: Combines transport and [[TLS]] 1.3 handshakes into **1 RTT** for new connections (vs 2–3 RTT for TCP + TLS) and **0-RTT** for resumed connections.
4. **Connection Migration**: Connections are identified by a **Connection ID (CID)** rather than the 4-tuple `(src IP, src port, dst IP, dst port)`, so switching from Wi-Fi to 5G doesn't drop in-flight requests.
5. **QPACK Header Compression**: Adapted from HTTP/2's HPACK to tolerate out-of-order stream delivery without blocking header decoding.
6. **Mandatory Encryption**: [[TLS]] 1.3 is built directly into QUIC (encrypting even transport metadata like packet numbers and ACKs), preventing middlebox ossification.

## Reliability Over [[UDP]]

[[UDP]] itself is unreliable and connectionless, but **QUIC** implements packet numbering, ACKs, loss detection, retransmission, and congestion control at the QUIC layer—making HTTP/3 just as reliable as TCP-based HTTP while remaining far more resilient on high-latency or lossy mobile links.

## Discovery & Fallback

Clients typically discover HTTP/3 support via the `Alt-Svc` response header (or DNS `HTTPS` record) on an initial [[HTTP2]] connection, then upgrade to QUIC over UDP port 443. If corporate firewalls block or throttle UDP 443, browsers automatically fall back ("Happy Eyeballs") to HTTP/2 over TCP.

> [!warning] Failure modes & trade-offs
> - **UDP 443 blocked or rate-limited**: many enterprise firewalls block outbound UDP 443, so servers and [[CDN|CDNs]] must always keep [[HTTP2]] over [[TCP]] as a fallback.
> - **Higher server CPU cost**: user-space QUIC + per-packet UDP processing lacks decades of NIC/kernel TCP offloading (GSO/GRO adoption is improving, but CPU cost per Gbps is still higher than tuned TCP).
> - **0-RTT replay risk**: just like [[TLS]] 1.3 0-RTT, QUIC 0-RTT early data can be replayed—never allow non-idempotent requests ([[API Idempotency]]) in 0-RTT.

## Flashcards

HTTP/3:::Latest [[HTTP]] version running over QUIC/[[UDP]] with built-in [[TLS]] 1.3, independent streams (no TCP Head-of-Line blocking), QPACK header compression, and Connection ID migration.

How does QUIC keep a connection alive when a mobile phone switches from Wi-Fi to cellular?::QUIC identifies connections by a ==Connection ID (CID)== rather than the IP/port 4-tuple, enabling seamless connection migration without re-handshaking.

HTTP/2 uses ==HPACK== over [[TCP]], whereas HTTP/3 uses ==QPACK== over QUIC/[[UDP]] so out-of-order streams don't block header decoding.

### Why can HTTP/3 outperform HTTP/2 dramatically on lossy mobile networks? #card

On a lossy network, a single lost packet in [[HTTP2]] stalls *all* multiplexed streams on the shared [[TCP]] connection. In HTTP/3 (QUIC), streams are independent so packet loss only delays the affected stream, and combined 1-RTT/0-RTT handshakes + Connection ID migration avoid reconnect stalls.

## Related
- [[HTTP]]
- [[HTTP2]]
- [[UDP]]
- [[TCP]]
- [[TLS]]
- [[CDN]]
