**Introduction:**
HTTP/1.1 (RFC 9112) was introduced in 1997 as an improvement over HTTP/1.0 and became the backbone of the web and [[REST]] APIs for over two decades.

**Key Features:**
1. **Persistent Connections (`Connection: keep-alive`)**: Connections stay open for multiple sequential request/response cycles, avoiding a fresh [[TCP]] + [[TLS]] handshake on every request.
2. **Pipelining**: Allowed sending multiple requests before receiving responses, but responses still had to return strictly in order—broken by proxies and Head-of-Line blocking, so disabled in modern browsers.
3. **Chunked Transfer Encoding**: Streams response bodies in length-prefixed chunks so the server can start sending before knowing the total `Content-Length`.
4. **Caching**: Rich cache validators (`ETag`, `If-None-Match`, `Cache-Control`) used heavily by browsers and [[CDN|CDNs]].
5. **Host Header & Content Negotiation**: Enabled virtual hosting (multiple domains on one IP) and format negotiation (`Accept`, `Accept-Encoding`).

**Limitations:**
1. **HTTP-level Head-of-Line Blocking**: Only one request can be served at a time per connection; a slow response stalls everything queued behind it.
2. **Connection Overhead**: Browsers open ~6 parallel [[TCP]] connections per origin (and historically used domain sharding) to fake concurrency, wasting sockets, memory, and TLS handshakes.
3. **Uncompressed Redundant Headers**: Every request resends verbose text headers (`Cookie`, `User-Agent`, `Authorization`) in full.

> [!warning] Failure modes & trade-offs
> - **HTTP Request Smuggling**: because HTTP/1.1 is text-delimited and allows both `Content-Length` and `Transfer-Encoding: chunked`, disagreements between a [[Reverse proxy]] and origin server over message boundaries allow attackers to smuggle hidden requests. Prefer [[HTTP2]] end-to-end or strict proxy normalization.

## Flashcards

HTTP/1.1:::Text-based [[HTTP]] version over [[TCP]] introducing persistent `keep-alive` connections, chunked transfer encoding, `Host` headers, and rich caching—but limited to one active request per connection.

Why do browsers open ~6 parallel [[TCP]] connections per domain under HTTP/1.1?::To work around ==HTTP-level Head-of-Line blocking==, since only one request/response can be in flight per HTTP/1.1 connection.

What causes HTTP Request Smuggling in HTTP/1.1 proxy chains?::Ambiguous framing when a [[Reverse proxy]] and backend disagree on `Content-Length` vs `Transfer-Encoding: chunked` message boundaries.

## Related
- [[HTTP]]
- [[HTTP2]]
- [[HTTP3]]
- [[TCP]]
- [[Reverse proxy]]
- [[REST]]
