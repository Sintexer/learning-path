---
flashcards:
  q-a9k9: { nid: 1795356715748, hash: u847eduq, sync: cdkvqrek }
  q-74zg: { nid: 1795361677058, hash: 25gsn4zi, sync: nc7fzwe4 }
  q-6cwu: { nid: 1795356715816, hash: b3jm82sf, sync: d8444isc }
---

The cluster's internal [[DNS]] service (backed by **CoreDNS**) runs as a [[K8S Deployment]] fronted by a `kube-dns` [[K8S Service]] with a static ClusterIP (e.g., `10.96.0.10`). `kubelet` injects that IP into every [[Pod]]'s `/etc/resolv.conf` so every container can automatically discover services by name without hardcoding IPs.

## DNS Naming Convention

Every [[K8S Service]] automatically receives a DNS record:

```text
<service-name>.<namespace>.svc.cluster.local
```

- **Regular (`ClusterIP`) Service**: resolves to the Service's single virtual `ClusterIP` (load-balanced at L4 by `kube-proxy`).
- **Headless Service (`clusterIP: None`)**: returns `A`/`AAAA` records for **every ready Pod IP** directly—used by StatefulSets (e.g., `pod-0.postgres.prod.svc.cluster.local`) and client-side [[gRPC]] [[Load Balancing]].

## Search Domains & the `ndots:5` Trap

By default, Kubernetes writes `/etc/resolv.conf` inside every Pod as:

```text
nameserver 10.96.0.10
search prod.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

Because `ndots:5` says *"any hostname with fewer than 5 dots is not an FQDN,"* a lookup for an **external** domain like `api.stripe.com` (only 2 dots) tries **4 wasted internal searches first**:
1. `api.stripe.com.prod.svc.cluster.local` (NXDOMAIN)
2. `api.stripe.com.svc.cluster.local` (NXDOMAIN)
3. `api.stripe.com.cluster.local` (NXDOMAIN)
4. `api.stripe.com.` (finally hits upstream!)

With dual-stack IPv4 (`A`) + IPv6 (`AAAA`), that multiplies a single external DNS call into **up to 8–10 UDP packets**!

> [!warning] Failure modes & trade-offs
> - **`ndots:5` DNS amplification & `conntrack` drops**: high-QPS services calling external APIs can flood CoreDNS and exhaust Linux `nf_conntrack` tables (causing mysterious 5-second DNS timeouts due to Linux UDP conntrack race conditions). Fix by setting `dnsConfig: { options: [{ name: "ndots", value: "2" }] }`, using trailing-dot FQDNs (`api.stripe.com.`), or deploying **NodeLocal DNSCache** (per-node caching DaemonSet over TCP to CoreDNS).
> - **CoreDNS single-node outage**: always run >= 2 CoreDNS replicas with `PodDisruptionBudget` and anti-affinity across nodes/AZs.

## Flashcards

What is the full cluster DNS name format for a [[K8S Service]]?::`<service-name>.<namespace>.svc.cluster.local`
^q-a9k9

How does DNS resolution differ between a normal `ClusterIP` Service and a **Headless Service** (`clusterIP: None`)?::A normal Service resolves to one virtual ==`ClusterIP`==; a Headless Service returns ==individual ready Pod IPs== (used for StatefulSets and client-side [[gRPC]] load balancing).
^q-pi8u
^q-74zg

### Why does Kubernetes's default `ndots:5` setting cause DNS amplification and latency spikes for external API calls? #card

Any hostname with fewer than 5 dots (e.g., `api.stripe.com`) is queried against all cluster search domains (`<ns>.svc.cluster.local`, `svc.cluster.local`, `cluster.local`) before trying the root domain—multiplying each lookup by 4–8x (with A + AAAA). Mitigate with `ndots:2`, trailing dots on external FQDNs, or **NodeLocal DNSCache**.
^q-6cwu

## Related
- [[DNS]]
- [[K8S Service]]
- [[K8S Core Containers]]
- [[Service discovery]]
- [[Kubernetes]]
- [[gRPC]]
