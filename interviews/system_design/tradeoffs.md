# Trade Offs

1. RDBMS vs Non-RDBMS
2. Distributed File System vs Database vs Object Store
3. gRPC vs REST
4. Protobuf vs Avro vs JSON vs Flatbuffer


## RDBMS vs Non-RDBMS (Key-Value store)
5 params to consider
1. Scalability
2. Performance
3. Data Model Complexity/Schema flexibility (Application code complexity)
4. Eventual Consistency
5. Data Locality for Queries

### RDBMS - SQL style
Data Model is schema based e.g. Employee having attributes like address, designation (which could be separate schemas as well). And query model is primary key + secorndary key or a combination with filtering criterias (like where designation = VP). These models require 
- Requires us to define all the attributes in the beginning, hard to update.
- Row based storage, requires all the attributes for all the rows.
- Consistency between diff models over availability
- Complex machinary to compute these queries.
**Pros**
1. **Strong Consistency:** Ensures ACID properties (atomicity, consistency, isolation, durability). All data changes are consistent and easily queried.
2. **Data Integrity:** Enforced relationships and constraints (e.g., foreign keys) maintain data integrity.
3. **SQL Queries:** Complex queries, joins, and aggregations are well-supported, which is useful for generating reports or analytics (e.g., querying user messages across channels).
4. **Mature Ecosystem:** Well-established tools for backup, replication, and monitoring.

**Cons:**
1. **Scalability Issues:** Relational databases struggle with horizontal scaling, especially with large-scale applications. Vertical scaling (increasing server resources) can only go so far.
2. **Write Performance Bottlenecks:** As user and message data grows, write-heavy operations (like inserting messages) can slow down, especially if there's heavy traffic (e.g., a message per second).
3. **Complexity with Large Data Volumes:** Storing and querying massive amounts of data, such as chat histories, can result in performance degradation due to the complexity of SQL joins or long-running queries.
4. Acheiving scalability and elasticity has been a huge challenge for Relational databases. Vertical scaling is possible, but its hard to scale them horizontally. 
* Relational databases are designed to run on a single server in order to maintain the integrity of the table mappings and avoid the problems of distributed computing. With this design, if a system needs to scale, customers must buy bigger, more complex, and more expensive proprietary hardware with more processing power, memory, and storage. 
* Today some techniquess have evolved which help scale RDBMS, using **Leader** and **follower** approach, where followers are additional servers that can handle parallel processing and replicated data. 
* Data can be sharded, in-memory processing, better use of replicas
* But ultimately there seems a single point of failure
* Also replication technologies chooses consistency over availability
* Although many advances have been made in the recent years, it is still not easy to scale-out databases or use smart partitioning schemes for load
balancing.
* The enhancements to relational databases also come with other big trade-offs as well. For example, when data is distributed across a relational database it is typically based on pre-defined queries in order to maintain performance. In other words, flexibility is sacrificed for performance.
* Additionally, relational databases are not designed to scale back down—they are highly **inelastic**. Once data has been distributed and additional space allocated, it is almost impossible to “undistribute” that data.

**Use Case Fit:**
- Ideal if you need strict consistency and complex queries across relational entities (e.g., joining user data with messages).
- Suitable for relatively lower traffic or small-to-medium scale applications, or where data integrity is a high priority.



### Non RDBMS - Key Value or No-SQL Style
Data model is key-value based. e.g. Id -> to a an object or an employee mapping.
- Attributes can be added to Employee object at will. Different objects of the same kind (Employee) could have diff attributes.
- scale-out “horizontally,” meaning that they run on multiple servers that work together, each sharing part of the load.
- Elastic: its easy to add or removes nodes based on the load.
Many services shoping cart, session management, sales rank, product catalog - need only primary key access, using RDBMS would lead to
inefficiencies and limit scale and availability. e.g. Dynamo provides a primary key interface. 
* Dont need complex quering and management functionality provided by RDBMS. The extra functionality requires expensive hardware and skilled personnel

###### NoSQL Document Store (e.g., MongoDB)
**How It Works:**
A NoSQL document store saves data as JSON-like documents, which can be nested and vary in structure. Collections of documents can represent different entities, such as Users, Channels, and Messages.
Example collections:
- Users Collection: stores user data in documents, each representing a user.
- Channels Collection: stores metadata about channels.
- Messages Collection: stores messages and can include embedded references to users and channels.

**Pros:**
1. **Scalability:** NoSQL databases are typically designed for horizontal scaling. They can distribute data across multiple nodes seamlessly, which is ideal for growing systems.
2. **Flexibility:** The schema-less nature allows for quick changes to data structures (e.g., adding fields to messages without migrations).
3. **Performance for Reads/Writes:** MongoDB, for example, is optimized for fast reads and writes, especially for unstructured or semi-structured data.
4. **Document Embedding:** You can embed related data (e.g., message history in the user document) to reduce the number of queries required.

**Cons:**
1. **Eventual Consistency:** Most NoSQL databases prioritize availability and partition tolerance (as per the CAP theorem), which can result in eventual consistency rather than strict consistency. This could cause issues when you need strong transactional guarantees (e.g., ensuring a message is always written after a channel is created).
2. **Complex Queries:** While MongoDB and similar systems support querying, it can be harder to perform complex joins or aggregations compared to relational databases. You often need to design your schema to avoid such queries.
3. **Data Duplication:** If you embed documents (e.g., embedding messages in channel documents), it may lead to data duplication and inefficient updates.

**Use Case Fit:**
1. **Best for Scalability:** If you're expecting rapid growth and need to scale horizontally with ease.
2. **Good for Performance-Critical Applications:** When high throughput (e.g., sending and retrieving messages quickly) and low-latency operations are crucial.
3. **Flexible Data Models:** Ideal for systems with evolving or loosely defined schemas

###### Key-Value Stores (e.g., Redis, DynamoDB)
**How It Works:**
Data is stored in simple key-value pairs, where the key is unique, and the value could be a string, list, or more complex data structure (e.g., sets, hashes).
- For example, Redis can be used to store user session data, cached messages, or notification states.

**Pros:**
1. **Fast Access:** Extremely fast for read/write operations, which makes it great for caching and session management.
2. **Scalability:** Can handle massive numbers of read and write requests due to its lightweight nature and the ability to scale horizontally.
3. **Simplicity:** Simple to set up and use.

**Cons:**
1. **Lack of Complex Queries:** Key-value stores don't provide advanced querying or aggregation capabilities. Data modeling for complex relationships (e.g., querying all messages from a user across channels) is challenging.
2. **Persistence Limitations:** While Redis offers persistence options (e.g., RDB snapshots, AOF), it is primarily used as an in-memory store. Storing large amounts of persistent data in Redis may be impractical.
3. **Limited Data Model:** Not ideal for applications that need complex relationships between entities like users, channels, and messages.

**Use Case Fit:**
1. **Best for Caching and Session Management:** If you need extremely fast access to specific data, such as recent messages or user session info.
2. **Not Ideal for Long-Term Storage:** Redis is not well-suited for long-term message storage but is a great option for frequently accessed, time-sensitive data.

## Replica Synchronization
Many systems traditionally use Synchronous replica coordination in order to provide strongly consistent data accesss interface, but they have to tradeoff 
availability. 
- We could follow **EventualConsistent Model** (Optimistic Replication techniques). This might lead to conflict
  1) When to resolve Conflict (during read or write)
  2) Who performs the process of conflict resolution
      * Data Store  (Limited choices - Last writer wins)
      * Application (Morre vsibility)

## Distributed File System vs Database vs Object Store

#### File System
- Files are stored in hierarchical manner, system manages an index.
- But only on the names of the files not on the content of the files.
- Coupled with OS
- A file system is a software application that organizes and maintains files on a storage device. It manages the storage and retrieval of data.
- Support for complex transaction is difficult
- Allows random access to data and update of the file

#### Databases
- Storage engine - file system abstracted away (database could be above a particular file system). A database management system (DBMS) provides an 
abstract representation of data that conceals the specifics.
- A database management system, or DBMS, is a software application that allows you to access, create, and manage databases.
- The database is a software application and is not shipped as a part of the operating system (its a 3rd party software). It is more about 
organizing the data and implementing techniques to keep the data consistent and to have faster access to the data.
- Has support for complex transaction
- ACID properties
- For object storage -> have to overwrite the file/object.

#### DFS vs Object Storage
- DFS allows random read/write of object, Object store only allows entire read/replacement of object
- DFS hierarchial, object Store (key-value)
- DFS can support Eventual and Strong consistency, Object store mostly supports eventual consistency.
- DFS posix file system api (or feels like a normal FileSystem call), Object Store usually rest api.
- DFS probably not so great for millions of small files - metadata little tricky to manage
- Object store slightly cheaper than DFS per/Gbs.
- DFS: Posix file system vs ObjectStore: Rest Api
- DFS can act as a backend for data storage, good for random reads/writes.
- Object storage, on the other hand, is more suitable for acting as a repository or archive of massive volumes of large files and comes at a significantly lower price 
- Speed: Data retrieval with object storage is faster. As you’re dealing with chunks of unstructured data on an individual basis, there is no directory system to create a bottleneck.

Here are some considerations for when to use a **distributed file system vs. a database:** </br>

- **Data types:** If your data is primarily unstructured or semi-structured, such as files, images, videos, or log data, a distributed file system may be a better choice. On the other hand, if your data is primarily structured, such as customer records, financial data, or inventory data, a database may be a better choice.

- **Consistency requirements:** If you require strong consistency guarantees and ACID transactions, a database may be a better choice. Databases provide strong consistency guarantees and allow you to enforce data integrity constraints, such as unique keys, foreign keys, and referential integrity. Distributed file systems, on the other hand, typically provide weaker consistency guarantees, such as eventual consistency or causal consistency.

- **Scale requirements:** If you need to store and process large volumes of data that require horizontal scaling, a distributed file system may be a better choice. Distributed file systems are designed to scale horizontally across multiple nodes, allowing you to store and process large volumes of data. Databases can also scale horizontally, but they may require more effort to set up and maintain.

- **Access patterns:** If your application requires frequent and random read/write access to the data, a database may be a better choice. Databases are optimized for handling frequent and random read/write access, and provide efficient indexing and query processing capabilities. Distributed file systems are optimized for large-scale, sequential access to data.

Overall, the choice between a distributed file system and a database depends on the specific needs and requirements of your application. If your application requires storing and processing large volumes of unstructured or semi-structured data, and you don't require strong consistency guarantees, a distributed file system may be a better choice. If your application requires storing structured data with strong consistency guarantees and ACID transactions, a database may be a better choice.

##### Some examples where DFS is used ?
Some examples of how DFS (Distributed File System) is used in real-world applications:

Video streaming services: Video streaming services, such as Netflix and Hulu, use DFS to store and distribute video files across multiple servers. This allows for faster access to the files and ensures that the videos can be streamed without interruptions.

Online backup and disaster recovery: Companies that provide online backup and disaster recovery services often use DFS to store and distribute backup files across multiple servers. This helps to ensure that the files are safe and accessible in the event of a disaster or hardware failure.

Web content management: Content management systems, such as WordPress and Drupal, use DFS to store and manage files, such as images and videos, that are used on websites. This allows the files to be easily shared and accessed by multiple users.

Scientific research: Scientific researchers often use DFS to store and share large datasets across multiple servers. This allows for faster data access and processing, which is critical for many types of research.

File sharing and collaboration: DFS is commonly used in enterprise environments to enable file sharing and collaboration among employees. This allows multiple users to access and edit files simultaneously, which can help to increase productivity and efficiency.



## gRPC vs REST
There are two primary models for API design: RPC and REST. Regardless of model, most modern APIs are implemented by mapping them in one way or another to the same HTTP protocol. 
3 ways to use HTTP
1) REST
2) gRPC
3) OpenAPI


### gRPC
- Based on RPC Api construct that uses HTTP2.0 as its underlying transport protocol
- gRPC: addressable entities are procedures (data is hidden behind procedures)
- Stubs are generated - code generation and strict specification
- HTTP model is hidden 

#### Useful
- Microservices Enviroment: low latency, 
- P2p real time communication: gRPC has excellent support for bi-directional streaming. gRPC services can push messages in real-time without polling
- Ployglot environment: diff languages 
- Network constrained environments: gRPC messages are serialized with Protobuf, a lightweight message format. A gRPC message is always smaller than an equivalent JSON message.
- Inter-process communication (IPC): IPC transports such as Unix domain sockets and named pipes can be used with gRPC to communicate between apps on the same machine. 

#### Not so useful
- Not human readable
- No browser support
- While HTTP request is easier to make (Browser or curl), requires code generatation.
- It’s simple to write a bot that crawls the entirety of a REST API without metadata, similarly to the way a browser or a web bot can crawl the entire HTML web. You can’t do this with an RPC-style API, regardless of whether it’s described using gRPC or OpenAPI, because RPC gives each entity type a different API that requires custom software or metadata to use it. I
- Does not define a mechanism to do partial updates (HTTP define a PATH method for partial updates)
- Does not define a standard mechanism to prevent data loss when 2 clients update the same resource at same time. HTTP defines standard Etag and If-Match headers for this purpose

### HTTP
- Addressable entities are data entities (called resources), behaviour is hidden behind data - CRUD.
- usually following REST standard for API definitions
- Browser supported.
- JSON schema model

#### Useful
- Usually used to expose an end point to external clients
- More standard tools for HTTP/REST
- Simple browser or curl command
- apis are proxied to add security features/input validations, parsing or modifying the headers, its hard to proxy gRPC (though Envoy supports this)

#### Not so useful
- HTTP with json model, 
  - json is hard to standardize - might break backward compatibility (protobuf on other hand supports backward compatibility
  - JSON model is verbose and less efficient

| Feature	| gRPC  |	HTTP APIs with JSON |
| ------  | ----- | ------------------- |
Contract	| Required (.proto)	| Optional (OpenAPI)
Protocol	| HTTP/2 |	HTTP
Payload	  |Protobuf (small, binary)	| JSON (large, human readable)
Prescriptiveness	| Strict specification	| Loose. Any HTTP is valid.
Streaming	| Client, server, bi-directional	| Client, server
Browser support |	No (requires grpc-web)	| Yes
Security	| Transport (TLS)	| Transport (TLS)
Client code-generation	|Yes |	OpenAPI + third-party tooling

## Protobuf vs Avro vs JSON vs Flatbuffer
### Protobuf
- By Google
- Binary serialization format, Binary serialization is faster than plain text serialization
- All the fields are optional
- Backward compatible
- Need proper tools to analyze this 
- **Schema management **: is much simpler, though there is extra boilerplate code cost we need to pay, but it essentially separates the s(d)erialization logic from domain
- Great support for multiple languages - Ruby, Java, python
- Good for complex modelling

### JSON
- Open shared file format (Javascript object notation)
- Human Readable
- Plain text, easy to read but not as compressable as binary format and requires more storage.
- Already many tools available
- 

### AVRO
- Apache Avro
- It uses JSON for defining data types and protocols, and serializes data in a compact binary format
- Need to provide the schema to reader and writer for deserialization. Schema registry is useful
- **Schema Management:** Avro can leak into your domain. Some constructions like e.g. a map with keys that are not strings are not supported in Avro model. When the serialization mechanism is forcing you to change something in your domain model — it’s not a good sign.
- have great support in Java
- Great for simple modelling
- Unified storage of schema and data, self-describing messages, no need to generate stub code (support for generating IDL)
- RPC calls exchange schema definitions during the handshake phase
- Schema definitions allow to define the ordering of the data (which will be followed when serializing)

### Flatbuffer
- By facebook
- Memory efficient, zero copy
- Would allow you to read specific fields without having to deserialize the entire object.

#### Schema Evolution
- Backward compabitibility is needed for reading older version of events
- Forward compatibility is needed for rolling updates



|Type  | Proto | Avro | JSON | Flatbuffer |
| -----| ------| ---- | -----| ---------- |
| Format| binary| Json but binary serialization | plain text | 
| Serialization Speed | Almost same as avro | Almost same as proto | Json is slow (plain text serialization) | 
| Size | Comparable with Avro | Comparable with proto | Much larger | 
| Compressable| yes | yes | not really | 
| Schema Evolution | Backward and forward compatibility (also fields are optional) | compatible (default values) slightly challenging than proto | Code needs to be updated to handle the json schema updates |
| Schema Management | Supports all data types | good for simple types, some complex types makes it challenging and info spills into domain |
| Language | Java, Python, Ruby, Go, Rust, CPP | Mostly Java & CPP | |
