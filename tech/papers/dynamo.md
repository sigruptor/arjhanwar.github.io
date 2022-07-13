# Dynamo - Highly Available Key Store

![Dynamo Paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)

## Introduction

 Reliable meaning - even if any components fail, it should still work. All the components are prone to errors and failures and also causes significant financial consequences.
And scale - to be allowed to have more capacity to support continuous growth. And at large scale, few components will fail every now and then. Failure handling to be the normal case w/o impacting performance and availability.

Problem

* Highly available - CAP - sacrifice consistency under certain failure scenarios.
* Application specific control of performance, durability and consistency
* Makes extensive use of Object versioning and application assisted conflict resolution.
* Amazon scale (3 million checkout/day, 1000s of concurrent users)

Amazon needed a highly available system for shopping cart use case. Dynamo is used to manage state of services with high availability requirements and need tight control over tradeoffs bw reliability, availability, cost-effectiveness and performance. It allows application designers to configure their data store appropriately based on the tradeoffs.

*Why not RDBMS:*
Many services shoping cart, session management, sales rank, product catalog - need only primary key access, using RDBMS would lead to inefficiencies and limit scale and availability. Dynamo provides a primary key interface. 

* Dont need complex quering and management functionality provided by RDBMS. The extra functionality requires expensive hardware and skilled personnel
* Also replication technologies chooses consistency over availability

## *Requirements*

1. *Query Model:* Key access uniquely identified by key, state stored as binary objects (blobs). Applications that need to store objects that are relatively small (1MB)
2. *ACID -* ACID is expensive dynamo trades weaker Consistency. 
3. *SLAs -* 99.9 percentile - getting even higher percentile decision is not made based on cost-benefit analysis - significant increase in cost to improve performance that much.
4. Configurable performance and consistency tuning. Give services the control system properties such as durability and consistency, and let them to choose their tradeoffs b/w functionality, performance and cost effectiveness.

## *Design Considerations*

1. *Consistency:* Strong consistency is expensive and comes at the cost of availability. Optimistic replication where changes are allowed to propagate to replicas in background. It leads to conflicting views, when to resolve them and who resolves them. 
    *When: *during read, people should be able to add things to their cart so writes should be always available. 
    *Who: *Can be done by either application (has more control and context) or data store (if done here, it has limited options, like Last Writer wins)
2. *Incremental Scalability:* scale out hosts with minimal impact on operators and system
3. *Symmetry:* All nodes should be identical and have similar resposibilites as peer. It simplifies provisioniing and maintenance.
4. *Decentralization:* Decentralized peer to peer system.
5. *Heterogeneity:* Some servers powerful i.e. work distribution proportional to server capabilities.

## *System Architecture*

1. *Interface:* put(key, value, context), 
    context encodes the system metadata about the object that is opaque to caller and includes info such as version of object. Context is stored along with the object such that system can verify the validity of context obj from put request.
2. *Partitioning Algorithm: Consistent Hashing* and incremental stability.
3. [Image: image.png]
    1.  One node responsible for range of keys
    2. Addition or deletion of nodes only affects immediate neighbors
    3. Virtual Nodes: For uniform distribution, each node gets assigned to multiple points in the ring.
        1. number of virtual nodes is decided based on capacity accounting for heterogeneity in infrastructure
4. *Replication:* Each key is replicated N times (N configured per instance).
    1. Each key gets a coordinator node which is responsible for locally storing it and as well as storing it on N successor nodes. 
    2. Key has a preference list - node list containing data (N different physical hosts)
    3. How do they create this preference list ?
    4. System is designed in such a way that every node can determine which nodes should be in this list for any particular key.
5. *Data Versioning:* Asynchronously updates all the replicas
    1. New versions subsume previous versions. (syntanctic reconcilliation) .
    2. or client have to resolve the conflicts in case of multiple branches of an object. (Add cart, delete from a cart)
    3. High Availability for writes - Vector clocks (node, count) with reconciliation during reads - version size is decoupled from updates reads.
    4. Vector clocks (node, count).. conflicts can be resolved, vector clock size limit is set.
6. *Execution of get and put: ** *Each request has a preference list (N hosts)
    1. to have a load balancer which will forward the request to a node…and if it is not in N list, it will be forwarded (multiple hops - latency goes up)
    2. Partition aware client library that routes to appropriate coordinator nodes. - Lower latency
    3. Follows the quorum pattern,… R+W > N...... and for better latency R and W < N
    4. Put -> W-1 writes - success
    5. Get -> at least R results , divergent versions are reconsiled.
7. *Handling Failures - Hinted Handoff:* Follows sloppy quorum; all reads and writes are perforrmed on first N Healthy nodes.
    1. Instead of waiting for a temp failed node - it sends the request to other node which stores information in another local store, once the node recovers, the partition is synced and deleted from other node. 
    2. W can be set to 1 but for higher durability it is set to more than 1.
    3. Also Data is replicated to multiple data centers to deal with failure scenarios - data center crashes etc.
8. *Handling Permanent failures - Replica Sync -* hinted handoff works when system churn is low.
    1. Hinted handoff node goes down before sending the update. Anti-entropy (replica syn) protocol to keep replicas synchronized.
    2. Merkle tree (hash of key is stored at the leaves and parent stores hash of its children) ..each node stores ranges of keys..
    3. [Image: image.png]
    4. This enables minimum data transfer between replicas
9. *Membership and failure detection*
    1. *Ring Membership*
        1. Node outages are transient and should not result in rebalancing of the partition assignment or repair of unreachable replicas.
        2. explicit mechanism to add and remove nodes from dynamo ring.
        3. Gossip based protocol propagates membership changes and maintains a eventually consistent view of membership. Hence each node has information about every other node’s key spaces
    2. *External discovery:* for initial seed, so that these are discovered via external means and are known to all the nodes.
    3. *Failure Detection:* Local view of failure is sufficient - if node A cant reach Node B…It will mark it as failed and send request to next replica.
        1. Only when a client request is made
        2. Gossip based failure view bw all the nodes..but with external discovery it was not needed. Local view of failures was sufficient.
10. *Adding removing Storage Nodes :* Bootstrapping, it is assigned a set of keys and the other nodes leave the control of those keys. Helps in uniform load distribution 

## Problem	Technique	Advantage
Partitioning	Consistent Hashing	Incremental Scalability
HA for writes	Vector clocks with reconciliation during reads	Version size decoupled from update rates
Handling Temp failures	Sloopy quorum and hinted handoff	Provides HA and durability guarantees when some replicas are not available
Recovering from permanent failures	Merkle trees	Syncronizes divergent replicas in background
Membership and failure detection	Gossip based membership protocol and failure detection.	Preserves symmetry and avoids having a centralized registry for storing membership and node liveness information.

Scalable and robust Solutions for 

* load balancing
* membership & Failure detection, 
* Failure recovery
* Replica Synchronization
* Overload handling
* State Transfer
* Concurrency and Job Scheduling 
* Request marshalling, request routing
* Monitoring and alarming
* Configuration management



## *Experiences and lessons learned*

1. Business logic specific reconciliation - shopping cart
2. Time based reconciliation - LWW
3. High performance read engine 

Users can choose what they want - W -> 1…but durability might suffer.

1. *Durability and performance:* Writes in the buffer - but if server crashes 
2. *Ensuring Uniform Load distribution:* Consistent hashing for uniform key distribution - uniform load distribution.
    1. High load - proper balance as all keys are accessed but during low load - only some keys are accessed hence improper balance
    2. Partitioning schemes
3. *Divergent versions - How many :* 1) Due to failures, data center failures, network partitions
    1. Concurrent writers - large number of concurrent writers to the single data item and multiple nodes end up coordinating the updates concurrently. But this only happens for bots not humans
4. *Client driven or server driven coordination*
    1. Loadbalancer sends the request to a random node coordinator. For read any node can be coordinator but for writes - coordinator has be from preferential list. Extrra hop
    2. Clients can keep the state from server and poll the information every 10 sec. 
        1. Server has to keep less state
        2. But client can remain outdated for 10 sec
5. *Foreground and background processing*
    1. Each node doing foreground (get and put) and background (hinted handoff, replica sync). There is resource contention bw them. 
    2. Admission control mechanism in place which is used by background tasks to reserve runtime slices of resources. 
    3. Admission control monitors the performance of foreground tasks and is employed to change the number of slices.
    4. Latencies for disk operations, failed database access due to lock contention, transaction timeouts, request queue wait times and compare it with threshold over last 60 seconds. 

Thoughts

* Too much work on client
* Too many concurrent users could happen in practice ?
* Hot partitions - small section of ring to be impacted

*Conclusion*

Well know techniques - 

* Data is partitioned and replicated using Consistent hashing
* Consistency facilitated by Object versioning
* Consistency among replicas - quorum-like technique and decentralized replica synchronization protocol
* Gossip based failure detection and membership protocol
* Addition/deletion of nodes w/o manual controls
* Decentralized system can be made to provide highly available system
* Eventually consistent
* Clients can configure W, R, N

