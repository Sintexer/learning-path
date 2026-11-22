---
flashcards:
  q-5khh: { nid: 1795356723991, hash: jm63iggu, sync: iea2z7zv }
  q-8s5w: { nid: 1795356724017, hash: hfmuws92, sync: k7ny55tm }
  q-tuae: { nid: 1795356724041, hash: ymbg5qxd, sync: qp4a6y7y }
  q-3u32: { nid: 1795356724067, hash: pckspaf4, sync: 3n72gqd5 }
  q-qmiw: { nid: 1795356724092, hash: dc4zvtcc, sync: amyi8k9s }
---

> AKA **Domain Name System**

**DNS** is the hierarchical, distributed naming system that translates human-readable domain names (e.g., `api.example.com`) into IP addresses and service metadata. It runs primarily over [[UDP]] port 53 (falling back to [[TCP]] port 53 for large responses, DNSSEC, or zone transfers) at [[OSI 7 layer]].

## Resolution Flow

1. **Local caches**: browser cache -> OS stub resolver cache (`/etc/hosts`, `systemd-resolved`).
2. **Recursive resolver** (ISP, `8.8.8.8`, or [[K8S DNS|CoreDNS]]): checks its cache; on a miss, walks the hierarchy:
   - **Root nameservers** (`.`) -> returns TLD nameserver (e.g., `.com`).
   - **TLD nameservers** (`.com`) -> returns authoritative nameserver for `example.com`.
   - **Authoritative nameserver** -> returns the final record + **TTL** (Time To Live).
3. Resolver caches the answer for `TTL` seconds and returns it to the client.

## Common Record Types

| Record | Purpose |
| --- | --- |
| `A` / `AAAA` | Maps domain name to IPv4 / IPv6 address |
| `CNAME` | Alias from one domain name to another (cannot sit at the zone apex) |
| `ALIAS` / `ANAME` | Provider-managed apex alias that resolves to `A`/`AAAA` at query time |
| `NS` | Delegates a zone to authoritative nameservers |
| `TXT` | Arbitrary text (SPF/DKIM/DMARC, domain ownership verification) |
| `SRV` | Host + port + priority/weight for [[Service discovery]] |
| `MX` | Mail exchange servers for a domain |

> [!warning] Failure modes & trade-offs
> - **DNS TTL caching traps**: lowering a TTL *during* an outage does not help clients already caching the old high-TTL record. Pre-lower TTLs (e.g., 60s) hours before planned migrations. Also beware runtimes like older JVMs that cache DNS indefinitely unless `networkaddress.cache.ttl` is configured.
> - **Negative caching**: `NXDOMAIN` responses are also cached (governed by the SOA `MINIMUM` field); creating a service *after* clients already queried it can stall resolution until negative TTL expires.
> - **DNS-based failover is slow**: ISPs and corporate resolvers often ignore very low TTLs (< 30s), making DNS a coarse failover tool compared to anycast IPs or L4/L7 [[Load Balancing]].
> - **[[K8S DNS]] `ndots:5` amplification**: in [[Kubernetes]], short hostnames trigger up to 4–5 search-domain lookups per query (doubled for A + AAAA), spiking CoreDNS load and latency unless FQDNs end with a trailing dot.

## Flashcards

DNS:::Domain Name System — hierarchical distributed system that resolves domain names to IP addresses and service records, primarily over [[UDP]] port 53.
^q-5khh

When does DNS switch from [[UDP]] to [[TCP]]?::When responses exceed the UDP payload limit (classic 512 bytes / EDNS0 limit), for DNSSEC-heavy responses, or for zone transfers (AXFR).
^q-8s5w

Why does lowering DNS TTL during an active outage fail to speed up failover?::Because resolvers that already cached the record will keep serving it until the ==previous TTL== expires.
^q-3u32
^q-tuae

### What is the difference between a `CNAME` record and an `A` / `ALIAS` record at the zone apex? #card

A `CNAME` aliases one name to another and per RFC cannot coexist with other records (`NS`, `SOA`), so it is illegal at the zone apex (`example.com`). An `A`/`AAAA` points directly to an IP, while a provider-specific `ALIAS`/`ANAME` resolves the target internally and returns synthesized `A`/`AAAA` records at the apex.
^q-qmiw

## Related
- [[K8S DNS]]
- [[UDP]]
- [[TCP]]
- [[CDN]]
- [[Load Balancing]]
- [[Service discovery]]
- [[URL]]
