## HTTP Evolution

### HTTP/1.0
- New connection for every resource
- For loading a web page, system/browser needed to setup new connections for all resources (html, js, css, images etc.), making it inefficient and slow

### HTTP/1.1
- Keep alives were introduced, so the same connection could be reused. 
- However, request are to be sent one-by-one 
- Request 1 to load html
- Request 2 is sent after response 1 is received and so on...
- Client application would usually establish 3 separate connections, which is a lot more resource consuming

### HTTP/2
- Notion of binary encoding - requests can seen as a collection of packets.
- Concept of stream - one connection can have multiple streams i.e. 3 streams could request 3 diff resources over the same connection
- HPACK- binary compression of headers was also introduced.
- Request prioritization (Streams have priorities associated) and better flow control
- Server could push data itself.
- However, there is TCP HEAD of Line blocking
  - The role of TCP is to deliver the entire stream of bytes, in the correct order, from one endpoint to the other. When a TCP packet carrying some 
of those bytes is lost on the network path, it creates a gap in the stream and TCP needs to fill it by resending the affected packet when the loss 
is detected. While doing so, none of the successfully delivered bytes that follow the lost ones can be delivered to the application, 
even if they were not themselves lost and belong to a completely independent HTTP request. So they end up getting unnecessarily delayed as TCP cannot
know whether the application would be able to process them without the missing bits. This problem is known as “head-of-line blocking”.

### HTTP/3 or QUIC
- QUIC Streams share QUIC connection, multiple independent streams over same connection.  QUIC streams are delivered independently such that in most cases 
packet loss affecting one stream doesn't affect others. This is possible because QUIC packets are encapsulated on top of UDP datagrams.
- QUIC is on top of UDP
- In User Space (while other n/w protocols are in kernel space), giving more control to app developer to update the protocol without updating the OS
version, essentially aiding in rapid deployment.
- HTTP streams can be mapped on top of QUIC streams, hence utilizing all benefits of HTTP/2 without head of line blocking
- Instead of HTTP/2 Header compression HPACK, quic uses QPACK for header compression
- QUIC connection is combining TCP+TLS handshake i.e. in the same step it establishes QUIC connection (with TLS)
**- Connection Migration**: Allows the migration of connection between WIFI and other carrier network.

#### Challenges with QUIC
- Many devices in between like Load balancers/firewall inspect and terminate TCP connection before passing the application data. This would be difficult
with QUIC, but QUIC does have connection ID concept, they can be used for routing by the load balancer without compromising QUIC's privacy and security guarantees.
- Many networks assume the ability to inspect TCP's state for indications of network problems (poor performance, high packet loss) or 
abusive traffic (data exfiltration, attacks). But quic blocks the ability to inspect the content, making it difficult to acheive the functionality. 
- However, since its in user space its gonna be challenging to have a common standard across diff applications/deployments.


### WebSockets
WebSockets can be a better choice when:

1. Real-time communication is the primary use case: WebSockets were designed specifically for real-time communication and are optimized for low latency and high throughput. If real-time communication is the primary use case, WebSockets may provide better performance and flexibility than HTTP/2.
2. Compatibility with older browsers is required: While HTTP/2 is supported by most modern web browsers, older browsers may not support it. WebSockets have broader support across a wider range of browsers, which can be important in some scenarios.
3. Full-duplex communication is required: While HTTP/2 supports bidirectional streaming, it is still based on the request/response paradigm of HTTP, which means that it may not be as efficient as WebSockets for full-duplex communication.

HTTP/2 can be a better alternative to WebSockets when:

1. Compatibility with existing infrastructure is important: HTTP/2 is built on top of the HTTP protocol, which means that it can be used with existing infrastructure and does not require separate ports or protocols.
2. Security is a concern: HTTP/2 includes several security features such as server push and header compression that can improve the security of web applications.
3. High volume data transfer is required: HTTP/2 uses a binary format for data transfer, which is more efficient than the text-based format used in WebSockets. This means that HTTP/2 can handle larger volumes of data and can provide better performance in high-traffic scenarios.

In HTTP/2, bidirectional communication is achieved through a single TCP connection, with both the client and server able to send multiple requests and responses at the same time. The requests and responses can be interleaved on the same connection, which allows for full-duplex communication. However, because HTTP/2 is still based on the request/response paradigm of HTTP, it may not be as efficient as WebSockets for full-duplex communication.

WebSockets, on the other hand, are specifically designed for full-duplex communication. When a WebSocket connection is established, the client and server can send messages to each other at any time, without having to wait for a request or response. This makes WebSockets more efficient than HTTP/2 for full-duplex communication.

**Q: for the server push if server is sending the data without client making the request, then isn't it similar to websockets**

While both server push in HTTP/2 and WebSockets allow the server to initiate the transfer of data to the client without the client first making a request, they differ in several important ways:
Protocol: WebSockets use a separate protocol, whereas server push is built on top of the HTTP/2 protocol.
Data transfer: With WebSockets, the client and server can send data to each other at any time, whereas with server push, the server can only initiate the transfer of data to the client.
Connection management: With WebSockets, a dedicated TCP connection is established between the client and server, whereas with server push, data is sent over an existing HTTP/2 connection.
Use cases: WebSockets are typically used for real-time, full-duplex communication, while server push is used to proactively send data to the client that is likely to be needed, such as CSS and JavaScript files, images, and other resources.

So while server push and WebSockets share some similarities in their ability to initiate data transfer from the server, they are designed for different use cases and work in different ways.


##### Resources
1. https://blog.cloudflare.com/http3-the-past-present-and-future/
2. https://www.wallarm.com/what/what-is-http-2-and-how-is-it-different-from-http-1
3. https://www.akamai.com/blog/performance/http3-and-quic-past-present-and-future
4. https://ably.com/topic/http-2-vs-http-3
5. https://ably.com/topic/websockets
