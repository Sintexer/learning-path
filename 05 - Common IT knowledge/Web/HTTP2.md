---
flashcards:
  q-4hk6: { nid: 1795356719617, hash: cpui2rc5, sync: yw6552m6 }
  q-squ2: { nid: 1795356719641, hash: zgvrvepg, sync: 5tvbt4pu }
  q-5dv2: { nid: 1795356719668, hash: 5hqbxu7x, sync: edxq3sdf }
  q-ue25: { nid: 1795356719694, hash: membk5ds, sync: 6dy7j84x }
---

**Introduction:**
HTTP/2 (RFC 9113) was published in 2015 to address the performance limitations of [[HTTP1.1]]. It evolved from Google's SPDY protocol and is the required transport for [[gRPC]].

**Key Features:**
1. **Binary Framing**: Splits messages into binary frames (HEADERS, DATA, RST_STREAM, WINDOW_UPDATE) instead of text, making parsing faster and eliminating text-delimited request smuggling.
2. **Multiplexing**: Interleaves multiple independent request/response streams over a single [[TCP]] connection simultaneously, eliminating **HTTP-level** Head-of-Line blocking.
3. **Header Compression (HPACK)**: Uses a shared dynamic table + Huffman encoding to compress repetitive headers across requests on the same connection.
4. **Per-Stream Flow Control**: Built-in `WINDOW_UPDATE` frames provide stream- and connection-level [[Backpressure]].
5. **Stream Prioritization**: Allows clients to hint which streams matter most (RFC 9218 extensible priorities). *(Note: HTTP/2 Server Push was later deprecated in browsers in favor of `103 Early Hints`.)*

**Limitations:**
1. **Head-of-Line Blocking at the [[TCP]] Level**: Because all multiplexed streams share one [[TCP]] byte stream, losing a single TCP packet stalls *every* stream on that connection until TCP retransmits it—often making HTTP/2 slower than HTTP/1.1 on lossy (>2% packet loss) mobile networks. Solved by [[HTTP3]] (QUIC).
2. **L4 [[Load Balancing]] Skew**: Because clients reuse one persistent connection, an L4 load balancer sends all requests from a client to a single backend; requires L7 load balancing (e.g., Envoy/NGINX) or client-side load balancing.

### gRPC and Browsers (`gRPC-Web`)

- **Initial Limitation**: [[gRPC]] relies on raw HTTP/2 framing features (especially HTTP/2 **trailers** for status codes and bidirectional streaming) that browser `fetch`/`XMLHttpRequest` APIs do not directly expose.
- **gRPC-Web**: A browser-compatible wire spec and JS client that translates gRPC calls over standard HTTP/1.1 or HTTP/2 through a proxy (typically Envoy) which converts them to native gRPC on the server side.

```javascript
import {grpc} from '@improbable-eng/grpc-web';
import {MyServiceClient} from './generated/my_service_pb_service';

const client = new MyServiceClient('http://localhost:8080');

client.myMethod(request, {}, (err, response) => {
  if (err) {
    console.error(err);
  } else {
    console.log(response);
  }
});
```

> [!warning] Failure modes & trade-offs
> - **HTTP/2 Rapid Reset (CVE-2023-44487)**: clients can open and immediately `RST_STREAM` millions of streams over a single TCP connection, bypassing traditional connection-based [[Rate Limiting]] and exhausting server CPU. Enforce `SETTINGS_MAX_CONCURRENT_STREAMS` and reset-rate limits at the [[Reverse proxy]] / [[API Gateway]].

## Flashcards

HTTP/2:::Binary, multiplexed [[HTTP]] protocol over [[TCP]] using HPACK header compression and per-stream flow control (transport for [[gRPC]]).
^q-4hk6

Why can HTTP/2 still suffer from Head-of-Line blocking even though it multiplexes streams?::Because all streams share a single ==[[TCP]] connection==; one lost TCP packet stalls delivery of *every* stream until retransmitted (solved by QUIC in [[HTTP3]]).
^q-5dv2
^q-squ2

### Why does HTTP/2 break naive L4 load balancing between microservices? #card

L4 load balancers distribute *connections*, not *requests*. Because an HTTP/2 (or [[gRPC]]) client multiplexes all RPCs onto one persistent [[TCP]] connection, all traffic pins to a single backend pod—requiring L7 load balancing ([[Reverse proxy]] / [[Service Mesh]]) or client-side load balancing.
^q-ue25

## Related
- [[HTTP]]
- [[HTTP1.1]]
- [[HTTP3]]
- [[gRPC]]
- [[TCP]]
- [[Load Balancing]]
- [[Backpressure]]
