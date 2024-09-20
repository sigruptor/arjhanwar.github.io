# KV Store

## Requirements
### Scale & Usage
1. Is it in-memory like redis/memcache or requires persistence storage
2. Is it specific to an application or generic solution
3. Do we need to support range queries

### Data Type & Operations
1. What kind of data we are going to write, is it blob or values?
2. Are there specific operations we need to support (e.g., atomic updates, transactions, range queries, etc.)?

### Consistency
1. Strong vs Eventual consistency ?
2. Do we need to worry about N/W Partitions ?

### Fault Tolerance & Durability
1. Do we design the system for node failures or is it for single node
2. Should writes be durable across restarts (i.e., should the data survive crashes)?

### Security & ACL
1. Do we need to support AuthN/Z, ACLs

### Performance:
1. What kind of latency is expected for read and write operations?
2. Is there a preference between write-heavy or read-heavy optimization?


## Considerations
### Data Persistence & Scalability
Since this needs to persist data and be highly scalable, the architecture should accommodate horizontal scaling.

**Storage Backends:** 
1. Distributed Database: We could leverage a distributed database (e.g., Apache Cassandra, or DynamoDB). This allows us to scale horizontally by adding more nodes and provides built-in mechanisms for replication, sharding, and fault tolerance. 
   **Trade-offs:** While distributed databases offer excellent scalability, they often come with more operational complexity (e.g., managing consistency, load balancing, and partitioning).

2. Log-Structured Merge Trees (LSM Trees) for Storage: This technique is commonly used in systems like RocksDB and LevelDB, providing efficient write performance.
   **Trade-offs:** LSM trees are highly performant for writes, but their read performance might be slightly lower compared to other systems. However, write amplification can occur during compaction processes.

Question:
Should we consider using an off-the-shelf distributed system like Cassandra, or build a custom storage layer with LSM trees and handle scaling ourselves?

### Consistency vs. Availability (Configurable)
Since we want the user to have a configurable option between consistency models, we need to make a choice between two well-known approaches:

#### Consistency Options:
**Strong Consistency (CP in CAP Theorem):** We can guarantee consistency by using mechanisms like Quorum-based writes/reads (majority voting across nodes). Each write will need to be acknowledged by the majority of replicas.

Trade-off: This may lead to higher latencies for writes, especially when we have network partitions or high load.

**Eventual Consistency** (AP in CAP Theorem): Writes are acknowledged as soon as a single replica stores the data. Consistency will be eventually achieved through background replication and reconciliation.

Trade-off: This is highly available and low-latency for writes, but users may see stale reads in the interim.

Question:
Do we want to support strong consistency only on writes, or also on reads (e.g., require reading from a majority of replicas)?

### Key-Value Pairs & Object Size
The objects can be blobs or other values but not very large. In this case, we can impose a size limit per value (e.g., 1-2 MB). This lets us optimize our architecture for small-to-moderate data sizes without needing complex chunking or large-object handling logic.

#### Key-Value Storage:
**Hash Partitioning:** We can partition data based on a hash of the key, which allows for distributing keys evenly across the cluster.

Trade-offs: Hash partitioning is efficient, but doesn’t naturally support range queries. This is fine for simple key-value lookups.

**Replication:** Replication will be important for fault tolerance. We can choose a replication factor to decide how many nodes hold copies of the same data.

Trade-offs: More replicas improve fault tolerance, but consume more storage and add write latency.

Question:
Should we impose a strict size limit per object to optimize for performance?

### Performance Tuning (Read-Heavy vs. Write-Heavy)
Since we want a generic solution that can handle both read-heavy and write-heavy workloads, we should make the architecture flexible to optimize either:

#### Tuning for Reads:
**Read Replicas:** For read-heavy workloads, we can allow for additional read replicas, which help distribute the load across the cluster.
Trade-offs: Adding more replicas increases storage costs and write latencies (as more nodes need to be updated).

#### Tuning for Writes:
**Write-Optimized Data Structures** (e.g., LSM Trees): Using write-optimized structures reduces the write overhead, especially for sequential writes.
Trade-offs: While write-optimized, these structures may have slightly slower read performance, requiring additional memory optimizations like in-memory indices.

Question:

Should we allow users to explicitly define read/write-heavy configurations, or can the system auto-tune based on the observed workload?

### High Availability and Fault Tolerance
To ensure high availability and fault tolerance, we can consider the following:

Strategies:
**Leader-Follower Replication:** A leader node handles writes, and followers replicate the data. This ensures consistency and high availability but may add latency in failover situations.
Trade-offs: While reliable, leader-follower replication may create bottlenecks if the leader is overloaded.

**Leaderless Consensus (Dynamo-style)**: Each node can accept writes, and we use quorum-based read/write repairs to ensure consistency.
Trade-offs: This approach is highly available and can tolerate failures more gracefully but makes handling conflicts more complex.

Question:
Do we need strict availability in case of partitioning, or is there a scenario where sacrificing availability for stronger consistency is acceptable?

### Security Features (Auth, Encryption, ACL)
For security, we can consider the following mechanisms:

Security Features:
**Authentication:** We can integrate with OAuth, JWT, or other token-based authentication methods to control access.

Trade-offs: OAuth or token-based authentication adds a slight overhead to each request, but it’s scalable and secure.

**Encryption:** Data should be encrypted both at rest and in transit. We can use SSL/TLS for network communication and AES-based encryption for data at rest.

Trade-offs: Encryption adds overhead during read/write operations, but is necessary for security.

**ACL (Access Control Lists)**: We can implement fine-grained access controls at the key-level or keyspace-level.

Trade-offs: Implementing ACLs increases complexity, and may affect performance if access rules need to be evaluated on each read/write.

Question:
Should ACLs be enforced at the key level (granular) or at the namespace/keyspace level (coarser)?
