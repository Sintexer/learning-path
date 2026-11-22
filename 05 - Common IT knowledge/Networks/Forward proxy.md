---
flashcards:
  q-rjfx: { nid: 1795356723916, hash: 2h3jwpwi, sync: 6rg69c7z }
  q-m5p2: { nid: 1795356723941, hash: it742ceh, sync: vfrytkx8 }
  q-6htt: { nid: 1795356723967, hash: shavqw7f, sync: 7abkzjvx }
---

A **forward proxy** is an intermediary server that sits between client devices and the internet (acting on behalf of the **client**). The destination server sees the proxy's IP address rather than the client's.

A forward proxy is commonly used for:

1. **Protecting clients** (hiding internal IPs and inspecting outbound traffic)
2. **Circumventing browsing restrictions** (routing through another network/region)
3. **Blocking access to certain content** (corporate web filtering, egress allowlisting, and auditing)
4. **Outbound caching & bandwidth reduction** (caching frequently fetched external artifacts/packages)

For [[HTTPS]] traffic, clients typically use the [[HTTP]] `CONNECT` method to establish an opaque [[TCP]] tunnel through the forward proxy so end-to-end [[TLS]] stays intact (unless the enterprise proxy performs TLS interception with a custom root CA).

> [!warning] Failure modes & trade-offs
> - **Single point of failure & egress bottleneck**: routing all outbound traffic from a [[Virtual Private Cloud|VPC]] or cluster through a forward proxy requires [[Horizontal scaling]] and [[Health Checks]] on the proxy tier.
> - **TLS interception breaks certificate pinning**: corporate forward proxies that MITM [[TLS]] break apps that pin server certificates (e.g., mobile apps, CLIs, package managers) unless the corporate root CA is trusted.

## Flashcards

Forward proxy:::An intermediary sitting in front of clients that forwards outbound requests to external servers on the clients' behalf (hiding the client from the server).
^q-rjfx

How does a forward proxy handle [[HTTPS]] traffic without decrypting it?::The client sends an [[HTTP]] `CONNECT` request to open a raw [[TCP]] tunnel to the target host, over which end-to-end [[TLS]] is negotiated.
^q-m5p2

### What is the core architectural difference between a forward proxy and a reverse proxy? #card

A **forward proxy** acts on behalf of **clients** (controlling/hiding outbound access to the internet), whereas a [[Reverse proxy|reverse proxy]] acts on behalf of **servers** (terminating [[TLS]], [[Load Balancing|load balancing]], and shielding origin backends).
^q-6htt

## Related
- [[Reverse proxy]]
- [[VPN]]
- [[HTTP]]
- [[TLS]]
- [[Zero-Trust Security]]
