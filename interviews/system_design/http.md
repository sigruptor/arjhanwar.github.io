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
- Protocol: WebSockets use a separate protocol, whereas server push is built on top of the HTTP/2 protocol.
- Data transfer: With WebSockets, the client and server can send data to each other at any time, whereas with server push, the server can only initiate the transfer of data to the client.
- Connection management: With WebSockets, a dedicated TCP connection is established between the client and server, whereas with server push, data is sent over an existing HTTP/2 connection.
- Use cases: WebSockets are typically used for real-time, full-duplex communication, while server push is used to proactively send data to the client that is likely to be needed, such as CSS and JavaScript files, images, and other resources.

So while server push and WebSockets share some similarities in their ability to initiate data transfer from the server, they are designed for different use cases and work in different ways.

**Q: Difference between long polling and server push ?**
Long polling and server push are both techniques used to achieve real-time, asynchronous communication between a client and server, but they differ in how they work.
Long polling involves the client making a request to the server, and the server holding onto the request until new data becomes available. When new data is available, the server responds to the client with the new data, and the client immediately makes another request to the server. This process continues indefinitely, allowing for real-time communication between the client and server.

Server push, on the other hand, involves the server proactively sending data to the client without the client first making a request. The server can push data to the client that is likely to be needed, such as CSS and JavaScript files, images, and other resources.
The main difference between long polling and server push is in how the data is transferred. Long polling requires the client to repeatedly make requests to the server, which can result in a large number of requests being made and processed. Server push, on the other hand, proactively sends data to the client, which can reduce the number of requests and improve efficiency.
Another difference is in how they handle network latency. With long polling, if the response from the server is delayed due to network latency or other factors, the client must wait for the response before making another request. With server push, data is sent proactively, so the client does not have to wait for a response.

Overall, long polling and server push are both techniques used for real-time communication between the client and server, but they work in different ways and are better suited to different use cases. Long polling is useful for situations where real-time communication is needed but the data transfer is infrequent, whereas server push is useful for situations where the server can proactively send data to the client that is likely to be needed.

**Q Some real world use cases for WebSockets**
WebSockets are widely used in a variety of real-world applications for building real-time, interactive, and collaborative web applications. Here are some examples of how WebSockets are used:
- Chat applications: WebSockets are commonly used for building chat applications. WebSockets provide low-latency, real-time, bidirectional communication between the client and server, which is essential for chat applications. Popular chat applications like Slack and Discord use WebSockets to provide real-time messaging.
- Multiplayer games: WebSockets are also commonly used for building multiplayer games. WebSockets provide a fast and efficient way to exchange game state and actions between players and the server. Popular multiplayer games like Agar.io and Slither.io use WebSockets to provide real-time gameplay.
- Real-time data visualization: WebSockets are useful for building real-time data visualization applications. For example, trading platforms use WebSockets to provide real-time updates of stock prices and other financial data. This allows traders to make informed decisions based on the most up-to-date information.
- Collaborative document editing: WebSockets are also useful for building collaborative document editing applications. WebSockets can be used to synchronize changes made by multiple users to a document in real-time. Google Docs and Office 365 use WebSockets to provide real-time collaboration features.


## Connection Options Deep Dive
### HTTP Connections:

**Basic HTTP:** Stateless protocol where each request-response cycle is independent. Not efficient for real-time communication since a new connection must be established for each request.

**HTTP/2:**
Supports multiplexing, allowing multiple requests and responses over a single connection. This can improve efficiency and reduce latency.
Header Compression: Reduces overhead, which can improve performance.
Still primarily a request-response model, but it can be more efficient than HTTP/1.1 for scenarios with many simultaneous requests.

### WebSockets:

Establishment: Begins with an HTTP handshake, which upgrades the connection to a persistent WebSocket connection.
Data Exchange: Once established, data is sent as frames, which is lightweight compared to HTTP requests. This enables low-latency, full-duplex communication.
Use Cases: Ideal for applications requiring real-time updates, like chat applications or notifications.

### Long Polling:

The client makes a request to the server, and the server holds that request open until there’s new information to send back.
After sending the response, the client immediately sends another request, effectively simulating a persistent connection.
Drawbacks: More overhead compared to WebSockets since it requires multiple HTTP requests and responses, leading to latency and higher resource consumption.

### Server-Sent Events (SSE):

One-way communication from the server to the client. The client establishes a connection and listens for messages.
Pros: Simpler to implement for real-time updates, and it automatically handles reconnections.
Cons: Not suitable for bidirectional communication, which can limit use cases.

#### Considerations for Choosing a Connection Method
Use Case: If you need real-time, bidirectional communication, WebSockets are your best option. For simpler one-way updates, SSE could suffice.
Scalability: Consider how many concurrent connections you need to support. WebSockets can be more efficient at scale, but you’ll need to ensure your infrastructure can handle the load.
Complexity: Evaluate the ease of implementation versus the requirements. HTTP/2 might be easier to integrate into existing services without substantial changes.


##### Resources
1. https://blog.cloudflare.com/http3-the-past-present-and-future/
2. https://www.wallarm.com/what/what-is-http-2-and-how-is-it-different-from-http-1
3. https://www.akamai.com/blog/performance/http3-and-quic-past-present-and-future
4. https://ably.com/topic/http-2-vs-http-3
5. https://ably.com/topic/websockets
