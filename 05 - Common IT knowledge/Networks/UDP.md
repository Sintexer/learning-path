> AKA **User Datagram Protocol**

**UDP** is a connectionless, lightweight, message-oriented transport protocol at [[OSI 4 layer]]. It adds only an 8-byte header (source port, destination port, length, checksum) on top of IP—no handshake, no delivery guarantee, no ordering, and no built-in congestion control.

## Common Use Cases

- **[[DNS]]** (port 53): tiny single-packet request/response where [[TCP]] handshake overhead would double latency.
- **QUIC / [[HTTP3]]**: builds custom reliability, per-stream ordering (eliminating TCP Head-of-Line blocking), [[TLS]] 1.3, and connection migration in user space on top of UDP.
- **Real-time media & gaming (WebRTC, VoIP, live video)**: a late retransmitted frame is worse than a dropped frame.
- **Metrics & telemetry (StatsD, syslog)**: fire-and-forget packets where dropping an occasional sample is preferable to blocking the app.

> [!warning] Failure modes & trade-offs
> - **Packet loss, reordering, and duplication**: applications using raw UDP must tolerate or handle all three themselves.
> - **No built-in congestion control**: naive high-rate UDP senders can starve TCP flows and trigger catastrophic packet loss across shared links.
> - **UDP amplification / DDoS**: because UDP has no handshake, source IPs are easily spoofed; exposed UDP services (DNS, NTP, Memcached) can be abused for reflection attacks.
> - **MTU fragmentation**: datagrams larger than Path MTU (~1,200–1,472 bytes) get fragmented at IP layer; losing *one* fragment drops the entire datagram. Keep UDP payloads under ~1,200 bytes (as QUIC does).

## Flashcards

UDP:::User Datagram Protocol — connectionless, best-effort datagram transport protocol at [[OSI 4 layer]] with minimal 8-byte overhead and no delivery, ordering, or congestion guarantees.

Why does [[HTTP3]] (QUIC) run over UDP instead of [[TCP]]?::To implement independent multiplexed streams, integrated [[TLS]] 1.3, and connection migration in user space—eliminating ==TCP Head-of-Line blocking== and ossified OS/middlebox TCP constraints.

### When should you choose UDP over TCP? #card

When low latency matters more than guaranteed ordered delivery (VoIP, gaming, live streaming), for tiny single-RTT lookups ([[DNS]]), or when building a custom transport protocol in user space (QUIC / [[HTTP3]]).

## Related
- [[TCP]]
- [[OSI 4 layer]]
- [[DNS]]
- [[HTTP3]]
