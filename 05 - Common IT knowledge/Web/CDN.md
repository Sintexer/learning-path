---
flashcards:
  q-dwx3: { nid: 1795356720043, hash: bvc4hgui, sync: y6bcdr9i }
  q-wv3e: { nid: 1795361711956, hash: dkvjvjat, sync: v5wnz2fq }
  q-44g5: { nid: 1795356720117, hash: ncpdvww9, sync: qwre724e }
  q-6xyk: { nid: 1795356720142, hash: 675ekcg2, sync: 9epmbk83 }
---

> AKA **Content Delivery Network**

A **Content Delivery Network (CDN)** is a globally distributed network of edge [[Reverse proxy|reverse proxy]] servers (Points of Presence / PoPs) that cache and serve content geographically close to end users, reducing latency and shielding origin servers from traffic spikes and DDoS attacks.

## Push vs Pull CDN

- **Pull CDN** (most common — CloudFront, Cloudflare, Fastly): the edge fetches an object from your origin on the first cache miss, caches it according to `Cache-Control` headers, and serves subsequent requests from the edge.
- **Push CDN**: you proactively upload static assets to the CDN before users request them (useful for large infrequent media drops or cold-cache avoidance).

## How Requests Reach the Nearest Edge

- **Anycast routing** (BGP): multiple PoPs advertise the same IP prefix; internet routers naturally send packets to the topologically nearest edge.
- **Geo-[[DNS]]**: authoritative DNS returns different edge IPs based on the client resolver's region.

## Caching & Invalidation Controls

- `Cache-Control: public, max-age=31536000, immutable` for content-hashed static assets (`app.8f3a1c.js`) — avoids expensive cache purges entirely.
- `s-maxage` overrides `max-age` specifically for shared caches (CDNs).
- `stale-while-revalidate` serves slightly stale content immediately while refreshing asynchronously in the background.
- `Vary` header splits cache entries by request header (e.g., `Accept-Encoding`); never `Vary` on high-cardinality headers like `Cookie` or `User-Agent`.

> [!warning] Failure modes & trade-offs
> - **Cache stampede (thundering herd)**: when a hot object expires, thousands of simultaneous misses can slam the origin unless the CDN supports **request coalescing** (collapsed forwarding) and **origin shield** (a mid-tier cache layer).
> - **Cache poisoning / private data leak**: accidentally caching a personalized response (`Set-Cookie` / `Authorization`) on a shared CDN serves one user's data to everyone—always mark user-specific responses `Cache-Control: private, no-store`.
> - **Slow global purge**: explicit URL invalidation takes seconds to minutes and costs money at scale; prefer immutable versioned filenames over purges.

## Flashcards

CDN:::Content Delivery Network — globally distributed edge [[Reverse proxy|reverse proxies]] that cache content close to users to cut latency, absorb DDoS traffic, and reduce origin load.
^q-dwx3

What is the cleanest strategy to avoid costly, racy CDN cache purges for static assets?::Embed a ==content hash== in the asset filename (e.g., `app.8f3a1c.js`) and serve it with `Cache-Control: public, max-age=31536000, immutable`.
^q-mtxs
^q-wv3e

Never cache authenticated or user-specific responses on a shared CDN without explicit `Cache-Control: private, no-store`. #card-reminder
^q-44g5

### How do request coalescing and an origin shield prevent a CDN cache stampede? #card

- **Request coalescing (collapsed forwarding)** holds concurrent cache-miss requests for the same key at an edge PoP and sends only *one* request upstream.
- **Origin shield** adds a centralized mid-tier cache between all global PoPs and the origin so N PoPs don't all miss to the origin simultaneously.
^q-6xyk

## Related
- [[Reverse proxy]]
- [[DNS]]
- [[HTTP]]
- [[HTTP3]]
- [[Load Balancing]]
- [[API Performance improvement]]
