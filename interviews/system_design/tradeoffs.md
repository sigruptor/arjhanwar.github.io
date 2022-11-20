# Trade Offs

1. RDBMS vs Non-RDBMS
2. Distributed File System vs Database vs Object Store
3. gRPC vs REST
4. Protobuf vs Avro vs JSON vs Flatbuffer


## RDBMS vs Non-RDBMS (Key-Value store)
3 params to consider
1. Application code complexity
2. Schema flexibility
3. Data Locality for Queries

### RDBMS - SQL style
Data Model is schema based e.g. Employee having attributes like address, designation (which could be separate schemas as well). And query model is primary key + secorndary key or a combination with filtering criterias (like where designation = VP). These models require 
- Requires us to define all the attributes in the beginning, hard to update.
- Row based storage, requires all the attributes for all the rows.
- Consistency between diff models over availability
- Complex machinary to compute these queries.

Acheiving scalability and elasticity has been a huge challenge for Relational databases. Vertical scaling is possible, but its hard to scale them horizontally. 
* Relational databases are designed to run on a single server in order to maintain the integrity of the table mappings and avoid the problems of distributed computing. With this design, if a system needs to scale, customers must buy bigger, more complex, and more expensive proprietary hardware with more processing power, memory, and storage. 
* Today some techniquess have evolved which help scale RDBMS, using **Leader** and **follower** approach, where followers are additional servers that can handle parallel processing and replicated data. 
* Data can be shareded, in-memory processing, better use of replicas
* But ultimately there seems a single point of failure
* Also replication technologies chooses consistency over availability
* Although many advances have been made in the recent years, it is still not easy to scale-out databases or use smart partitioning schemes for load
balancing.
* The enhancements to relational databases also come with other big trade-offs as well. For example, when data is distributed across a relational database it is typically based on pre-defined queries in order to maintain performance. In other words, flexibility is sacrificed for performance.
* Additionally, relational databases are not designed to scale back down—they are highly **inelastic**. Once data has been distributed and additional space allocated, it is almost impossible to “undistribute” that data.

### Non RDBMS - Key Value or No-SQL Style
Data model is key-value based. e.g. Id -> to a an object or an employee mapping.
- Attributes can be added to Employee object at will. Different objects of the same kind (Employee) could have diff attributes.
- scale-out “horizontally,” meaning that they run on multiple servers that work together, each sharing part of the load.
- Elastic: its easy to add or removes nodes based on the load.
Many services shoping cart, session management, sales rank, product catalog - need only primary key access, using RDBMS would lead to
inefficiencies and limit scale and availability. e.g. Dynamo provides a primary key interface. 
* Dont need complex quering and management functionality provided by RDBMS. The extra functionality requires expensive hardware and skilled personnel


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
