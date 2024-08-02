
**Introduction:**
HTTP/1.1 was introduced in 1997 as an improvement over the original HTTP/1.0. It became the most widely used version of HTTP for many years.

**Key Features:**
1. **Persistent Connections**: Connections are kept open for multiple requests/responses, reducing the overhead of establishing new connections.
2. **Pipelining**: Allows multiple requests to be sent out without waiting for the corresponding responses. However, it is rarely used due to head-of-line blocking issues.
3. **Chunked Transfer Encoding**: Supports sending data in chunks, allowing the server to start sending a response before knowing the total size.
4. **Caching**: Enhanced caching mechanisms to improve performance.
5. **Content Negotiation**: Allows clients and servers to negotiate the best representation of a resource.

**Limitations:**
1. **Head-of-Line Blocking**: Only one request can be outstanding on a connection at a time, leading to delays.
2. **Inefficient Use of Connections**: Multiple connections are often needed to achieve better performance, leading to increased resource usage.