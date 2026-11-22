> AKA **Internet of Things**

**IoT (Internet of Things)** refers to networks of physical devices (sensors, actuators, industrial controllers, wearables, smart meters) embedded with compute, firmware, and connectivity that collect telemetry or receive commands over a network.

## Systems Architecture Characteristics

- **Resource-constrained edge**: Limited CPU, RAM, flash storage, battery, and intermittent/high-latency cellular or LPWAN links.
- **Lightweight protocols**:
  - **MQTT**: Lightweight pub/sub protocol over [[TCP]]/[[TLS]] with QoS levels ([[At-most-once-delivery|QoS 0]], [[At-least-once delivery|QoS 1]], [[Exactly-once delivery|QoS 2]]) and Last Will & Testament (LWT) disconnect notifications.
  - **CoAP**: REST-like request/response protocol over [[UDP]] (DTLS) for ultra-constrained devices.
  - **[[HTTP]] / [[WebSocket]]**: Used on higher-powered gateways where firewall traversal matters most.
- **Cloud ingestion pipeline**: Devices connect to an edge/IoT broker → events stream into [[Kafka]] (decoupling ingestion spikes) → hot path ([[OLTP]] / alerts) and cold path (time-series DB / [[Data Warehouse]] / [[OLAP]]).
- **Device identity & security**: Each device should hold a unique hardware-backed X.509 certificate (mTLS) rather than a fleet-wide shared API key ([[Zero-Trust Security]], [[Least privilege principle]]).

Why is MQTT commonly preferred over plain HTTP for IoT telemetry?::MQTT maintains a persistent, low-overhead binary TCP connection with tiny headers, built-in pub/sub fan-out, configurable QoS delivery levels, and Last Will disconnect detection—saving battery and bandwidth compared to HTTP's per-request overhead.

> [!warning] Failure modes & trade-offs
> - **Thundering herd reconnect storms**: When a regional cell tower or broker restarts, millions of devices reconnecting simultaneously can DDoS the ingestion tier. Always implement exponential backoff with random jitter ([[Retry Pattern]]), [[Rate Limiting]], and [[Backpressure]].
> - **Fleet-wide bricking via OTA updates**: Pushing a buggy firmware update to all devices at once can permanently brick hardware in the field. Always use A/B dual-partition rollbacks, signed firmware images, and staged canary rollouts (1% → 10% → 100%).
> - **Out-of-order & duplicate telemetry**: Offline devices buffer readings and flush in bursts upon reconnecting; design ingestion consumers as [[Idempotent Consumer|idempotent consumers]] keyed by `(device_id, timestamp/seq)` and distinguish **event time** from **ingestion time**.

Always separate **event time** (device timestamp) from **ingestion time** (server receipt time) and make IoT ingestion pipelines ==idempotent== to handle offline buffering, retries, and duplicate deliveries.

## How do you prevent a fleet of IoT devices from taking down your backend after a network outage? #card

1. **Exponential backoff + jitter** on device reconnects ([[Retry Pattern]]) so devices do not synchronize into a thundering herd.
2. **Edge connection draining & [[Rate Limiting]]** at the IoT broker/gateway.
3. **Buffered ingestion via [[Kafka]]** to absorb bursty telemetry spikes and apply [[Backpressure]] without dropping data.

## Related

- [[TCP]]
- [[UDP]]
- [[TLS]]
- [[Kafka]]
- [[Delivery Semantics]]
- [[Idempotent Consumer]]
- [[Retry Pattern]]
- [[Backpressure]]
- [[Rate Limiting]]
- [[Zero-Trust Security]]
