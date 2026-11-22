---
flashcards:
  q-3zce: { nid: 1795356719091, hash: qubhh48e, sync: i2hdkh4r }
  q-6aj4: { nid: 1795356719117, hash: 8kxkg4wp, sync: 9v9m2pvz }
  q-z5cq: { nid: 1795356719142, hash: aychyd2p, sync: 8xydvb4n }
  q-5mn9: { nid: 1795356719175, hash: mtbm89mh, sync: xtsx33rp }
---

> AKA **Uniform Resource Locator**

A **URL** is a subset of [[URI]] that specifies **where a resource is located** and **how it can be retrieved** (its network access mechanism). Its standard anatomy is:

`scheme://host:port/path?query#fragment`

- **Scheme (Protocol)**: `https`, `http`, `wss`, `grpc`
- **Authority**: hostname (resolved via [[DNS]]) and optional port
- **Path & Query**: resource path and key-value parameters sent to the server
- **Fragment (`#...`)**: client-side anchor—**never sent to the server** in an [[HTTP]] request

> [!example]
> `https://www.google.com/search?q=example`

## Difference between URI and URL

All *URLs are URIs*, but *not all URIs are URLs*. A URI identifies a resource by location, name, or both. If it identifies a resource by location and retrieval mechanism, it's a URL. **Some URIs are not URLs because they identify a resource only by name (URNs) rather than by location.**

> [!hint]
> In typical web-discussion context, most people say URL even when they technically mean URI.

> [!warning] Failure modes & trade-offs
> - **Secrets in URL query strings**: query parameters are logged in plaintext by proxies, [[CDN|CDNs]], browser history, and `Referer` headers—never pass tokens, passwords, or PII in URLs; use `Authorization` headers or request bodies.

## Flashcards

URL:::Uniform Resource Locator — a subset of [[URI]] that specifies both *where* a resource lives and *how* to retrieve it (`scheme://host:port/path?query`).
^q-3zce

Which part of a URL is never sent over the network to the server in an [[HTTP]] request?::The ==fragment== (`#section`), which is processed strictly on the client.
^q-z5cq
^q-6aj4

Never place secrets, tokens, or PII in URL query strings because proxies, access logs, and browser history record URLs in plaintext. #card-reminder
^q-5mn9

## Related
- [[URI]]
- [[HTTP]]
- [[DNS]]
- [[REST]]
