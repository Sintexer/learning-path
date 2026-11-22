---
flashcards:
  q-vxvv: { nid: 1795356718791, hash: ye4dj23k, sync: zbgbp3rx }
  q-yc5i: { nid: 1795356718825, hash: 2wtji7v9, sync: 6nzsmu24 }
  q-dq4e: { nid: 1795356718842, hash: jwy5um7n, sync: agz5ixb2 }
  q-d9vg: { nid: 1795356718868, hash: eptq24k3, sync: r49dntbx }
  q-h5ix: { nid: 1795356718893, hash: 26dkwcg3, sync: 4qb86ku7 }
  q-2w7s: { nid: 1795356718918, hash: q5v3t5qz, sync: 5du87kq8 }
---

> AKA **gRPC Remote Procedure Call**

**gRPC** is a high-performance, open-source RPC framework (originally from Google) designed primarily for low-latency, strongly typed service-to-service communication in [[Microservices App|microservices]]. It runs over [[HTTP2]] and uses [[Protocol Buffers]] (Protobuf) as both its Interface Definition Language (IDL) and compact binary wire format.

## Four Communication Patterns

1. **Unary RPC**: classic single request -> single response (like [[REST]]).
2. **Server streaming**: client sends one request; server streams back a sequence of messages.
3. **Client streaming**: client streams a sequence of messages; server returns one summary response.
4. **Bidirectional streaming**: both client and server stream messages independently over a single [[HTTP2]] stream (with built-in flow control / [[Backpressure]]).

## Why Teams Choose gRPC Over [[REST]]

- **Compact binary serialization** ([[Protocol Buffers]]) -> smaller payloads and much faster CPU parsing than JSON.
- **Strict contract & code generation**: `.proto` files generate strongly typed clients/servers across languages (Java, Go, Python, Rust, C++) and enforce [[Backward Compatibility]] / [[Forward Compatibility]] via numbered field tags.
- **Native deadlines & cancellation**: clients propagate a **deadline** (`grpc-timeout`) across the entire call tree so downstream services stop wasted work when the caller has already timed out ([[Timeout Pattern]]).
- **Multiplexing & streaming**: inherits [[HTTP2]] multiplexing, HPACK header compression, and native streaming.

> [!warning] Failure modes & trade-offs
> - **L4 [[Load Balancing]] breaks gRPC**: because gRPC multiplexes all calls over a single persistent [[HTTP2]]/[[TCP]] connection, an L4 load balancer (including default [[K8S Service|Kubernetes ClusterIP Services]]) pins all traffic to one backend pod. Use L7 load balancing (Envoy / [[Service Mesh]]), client-side load balancing with headless services, or `max_connection_age`.
> - **Not natively browser-friendly**: browsers don't expose raw HTTP/2 trailers; browser clients need **gRPC-Web** via a proxy (Envoy) or a [[BFF]] / [[REST]] / [[GraphQL]] facade.
> - **Schema evolution discipline**: never reuse or change the type of a retired Protobuf field tag number—reserve removed tags (`reserved 4;`) to prevent silent data corruption during rolling deploys.
> - **Harder ad-hoc debugging**: binary payloads aren't human-readable in `curl`/browser DevTools; use `grpcurl`, Postman, or reflection + [[OpenTelemetry]] [[Distributed Tracing]].

## Flashcards

gRPC:::High-performance RPC framework using [[HTTP2]] for multiplexed transport/streaming and [[Protocol Buffers]] for strongly typed binary serialization.
^q-vxvv

What are the four RPC streaming patterns supported natively by gRPC?::==Unary==, ==server streaming==, ==client streaming==, and ==bidirectional streaming==.
^q-dq4e
^q-yc5i

Why does a default [[K8S Service]] (`ClusterIP` / L4) cause severe load imbalance for gRPC traffic?::Because gRPC multiplexes all RPCs over a single persistent [[HTTP2]]/[[TCP]] connection, so L4 connection-level load balancing pins all calls to ==one backend pod==; solve with L7 proxies ([[Service Mesh]]) or client-side load balancing.
^q-h5ix
^q-d9vg

### How does gRPC deadline propagation protect microservices from cascading overload? #card

Each gRPC call carries an absolute deadline (`grpc-timeout`) that propagates across downstream hops. When the deadline expires, downstream services receive cancellation signals and abort work instead of burning CPU on responses the caller will discard.
^q-2w7s

## Related
- [[HTTP2]]
- [[Protocol Buffers]]
- [[REST]]
- [[GraphQL]]
- [[WebSocket]]
- [[Load Balancing]]
- [[Service Mesh]]
- [[Backward Compatibility]]
- [[Timeout Pattern]]
