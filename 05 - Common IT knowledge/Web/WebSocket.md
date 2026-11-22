---
flashcards:
  q-4xfh: { nid: 1795356718667, hash: p3ntcjxd, sync: kderiuxh }
  q-pnqh: { nid: 1795361688881, hash: ic9hzwxu, sync: u3h4iayj }
  q-6u93: { nid: 1795356718742, hash: m7sccgyq, sync: jd37dh3r }
  q-43r4: { nid: 1795356718767, hash: 93v4fn4i, sync: hqmtetwr }
---

**WebSocket** (RFC 6455, `ws://` / `wss://`) is a full-duplex, persistent, bidirectional communication protocol over a single [[TCP]] connection at [[OSI 7 layer]]. It replaces inefficient [[Client polling pattern|HTTP polling]] for real-time features like chat, live trading/order-book feeds, collaborative editing, and notifications.

## Connection Lifecycle

1. **HTTP Upgrade Handshake**: client sends a standard [[HTTP1.1]] `GET` request with `Upgrade: websocket`, `Connection: Upgrade`, and `Sec-WebSocket-Key`.
2. **Protocol Switch**: server replies `101 Switching Protocols` with `Sec-WebSocket-Accept`; the underlying [[TCP]] socket now speaks lightweight binary/text WebSocket frames instead of HTTP.
3. **Full-Duplex Messaging**: either side can push frames independently at any time, with minimal framing overhead (2–14 bytes per frame) and built-in `Ping`/`Pong` control frames for liveness.

## WebSocket vs SSE vs [[gRPC]] Streaming vs [[Client polling pattern|Polling]]

| Mechanism | Direction | Transport | Best For |
| --- | --- | --- | --- |
| **Short/Long Polling** | Client -> Server | [[HTTP1.1]] | Simple fallback, low-frequency updates |
| **Server-Sent Events (SSE)** | Server -> Client only | [[HTTP]] | Live feeds, AI token streaming, notifications (auto-reconnects, proxy-friendly) |
| **WebSocket** | Full-duplex (bidirectional) | [[TCP]] (upgraded from HTTP) | Chat, multiplayer, live bidding/trading, collaborative docs |
| **[[gRPC]] Streaming** | Unary / Client / Server / BiDi | [[HTTP2]] | High-throughput service-to-service streaming with [[Protocol Buffers]] |

> [!warning] Failure modes & trade-offs
> - **Stateful horizontal scaling**: each WebSocket connection is pinned to one server instance. Scaling out ([[Horizontal scaling]]) requires a pub/sub backplane (e.g., [[Redis]] Pub/Sub, [[Kafka]], NATS) so instance A can route messages to a user connected to instance B.
> - **Rolling deployments & thundering herd**: terminating pods during a [[Zero-downtime deployment]] drops thousands of sockets simultaneously; if all clients reconnect at once without jittered backoff ([[Retry Pattern]]), they DDoS the cluster. Drain connections gradually and enforce client jitter.
> - **Slow consumers & memory OOM**: if a client stops reading, outbound socket buffers grow unbounded unless you enforce [[Backpressure]] (bounded per-connection queues, drop/disconnect slow clients).
> - **Proxy idle timeouts & auth**: L7 proxies/load balancers drop idle connections (often 60s)—send application or WebSocket `Ping`/`Pong` heartbeats, and remember browser WebSocket APIs don't support custom `Authorization` headers (use cookies or short-lived handshake tickets, not long-lived JWTs in query strings).

## Flashcards

WebSocket:::A full-duplex, persistent messaging protocol over a single [[TCP]] connection initiated via an [[HTTP]] `101 Switching Protocols` upgrade handshake.
^q-4xfh

When should you prefer Server-Sent Events (SSE) over WebSockets?::When communication is strictly ==server-to-client== (e.g., notifications, live feeds, LLM token streaming), because SSE works over standard [[HTTP]], auto-reconnects, and traverses proxies cleanly.
^q-qfi2
^q-pnqh

Always bound per-connection outbound buffers and disconnect slow WebSocket consumers to prevent server OOMs. #card-reminder
^q-6u93

### What breaks when you horizontally scale a WebSocket service, and how do you solve it? #card

WebSocket connections are stateful and pinned to individual server instances, so a message arriving at Instance A cannot directly reach a user connected to Instance B. Solve it by adding a shared pub/sub backplane ([[Redis]] Pub/Sub, [[Kafka]], or NATS) to fan out events across instances, and handle rolling restarts with gradual draining + jittered client reconnects.
^q-43r4

## Related
- [[HTTP]]
- [[TCP]]
- [[Client polling pattern]]
- [[gRPC]]
- [[Load Balancing]]
- [[Horizontal scaling]]
- [[Backpressure]]
- [[Zero-downtime deployment]]
