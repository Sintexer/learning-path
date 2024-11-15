Uses port 80.

See [[HTTPS]]

There are various implementations of HTTP protocol built in different years:

- [[HTTP1.1]] (REST, Web)
- [[HTTP2]] (gRPC)
- [[HTTP3]]

### Summary of Differences

| Feature                   | HTTP/1.1                         | HTTP/2                              | HTTP/3                              |
|---------------------------|----------------------------------|-------------------------------------|-------------------------------------|
| **Protocol**              | Text-based                       | Binary                              | Binary (over QUIC)                  |
| **Multiplexing**          | No                               | Yes                                 | Yes                                 |
| **Header Compression**    | No                               | Yes (HPACK)                         | Yes (QPACK)                         |
| **Server Push**           | No                               | Yes                                 | Yes                                 |
| **Transport Protocol**    | TCP                              | TCP                                 | UDP (QUIC)                          |
| **Head-of-Line Blocking** | Yes                              | Yes (at TCP level)                  | No                                  |
| **Connection Establishment** | Slow (TCP handshake + TLS handshake) | Faster (TCP handshake + TLS handshake) | Fastest (combined QUIC handshake)   |
| **Security**              | Optional (HTTPS)                 | Built-in (TLS)                      | Built-in (QUIC encryption)          |

### Examples of Adoption

1. **Google**: Google has been a major proponent of HTTP/2 and HTTP/3, with many of its services supporting these protocols.
2. **Facebook**: Facebook has also adopted HTTP/2 and is experimenting with HTTP/3 to improve performance.
3. **Cloudflare**: Cloudflare supports HTTP/2 and HTTP/3, providing these protocols to improve the performance of websites using its CDN services.
