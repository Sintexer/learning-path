**Introduction:**
HTTP/2 was published in 2015 to address the performance limitations of HTTP/1.1. It is based on Google's SPDY protocol. Used by [[gRPC]] technology.

**Key Features:**
1. **Binary Protocol**: Uses a binary format instead of text, making it more efficient to parse and less error-prone.
2. **Multiplexing**: Allows multiple requests and responses to be sent over a single connection simultaneously, eliminating head-of-line blocking.
3. **Header Compression**: Uses HPACK compression to reduce the size of headers, improving performance.
4. **Server Push**: Allows servers to push resources to the client proactively, reducing latency.
5. **Stream Prioritization**: Clients can prioritize streams, allowing more important resources to be delivered first.

**Improvements Over HTTP/1.1:**
1. **Reduced Latency**: Multiplexing and server push significantly reduce latency.
2. **Better Resource Utilization**: Single connection usage reduces the overhead of multiple connections.
3. **Improved Performance**: Header compression and binary framing improve overall performance.

**Limitations:**
1. **Complexity**: More complex than HTTP/1.1, requiring more sophisticated handling.
2. **Head-of-Line Blocking at TCP Level**: While HTTP/2 eliminates head-of-line blocking at the HTTP level, it still suffers from it at the TCP level.

#### Initial Limitation with gRPC and Browsers

**gRPC and HTTP/2:**
- **gRPC**: gRPC is a high-performance, open-source RPC framework that uses HTTP/2 for transport and Protocol Buffers for serialization.
- **Initial Limitation**: When gRPC was first introduced, it was primarily designed for server-to-server communication. This was because most browsers did not fully support HTTP/2 for non-HTTP traffic, making it challenging to use gRPC directly from web browsers.

#### Changes and Current State

**gRPC-Web:**
- **Introduction**: To address the limitation of using gRPC in web browsers, the gRPC team introduced **gRPC-Web**. gRPC-Web is a JavaScript client library that allows web applications to interact with gRPC services.
- **How It Works**: gRPC-Web works by translating gRPC calls into HTTP/1.1 or HTTP/2 requests that can be understood by web browsers. It uses a proxy server to convert these requests back into standard gRPC calls on the server side.
- **Adoption**: gRPC-Web has been widely adopted and is now a standard way to use gRPC in web applications.

**Example:**
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
