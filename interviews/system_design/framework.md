# System Design Framework

First need to figure out the requirements and assumptions. 
## Requirements/Assumptions
- Size of the data to be persistent in DB (persistent store)
- Number of users ?
- Consistency guarantees ? 
- Is Data loss okay or what could be the SLA ? - its expensive to improve a system beyond certain threshold without much signifant benefits.
- Security and Privacy ?

Any system in addition to the actual data persistence component, needs to have scalable and robust solution for 
1. Load balancing
2. Replication & Replica Synchronization
3. Membership and Failure Detection
4. Failure Recovery
5. Overload handling
6. State Transfer
7. Concurrency & Job scheduling
8. Request Marshalling & Request Routing
9. System Monitoring and Alarming


### Load Balancing
- Consistent Hashing & Virtual nodes

### Replication & Replica Synchronization
- Eventual Consistency -> much relazed - optimistic replication & conflict resolution
- Strong consistency -> Leader, follower
##### Data Versioning 
More details here [Replica Sync](./tradeoffs.md/#replica-synchronization)
- For conflict resolution 
  - at System level  (Last writer wins)
  - At Client level - more control
  - During Read/Write
  - Vector clocks
  - R, W quorum (R+w>N)

### Failure Recovery
#### Temporary Failures
- Hinted Handoff (separate DB to store temp state)
#### Permanent Faliures
- Replica Synchronization
- Merkle Tree
  - hash of keys at the leaves, parent is hash of children
  - Saves on Data transfer and computing the diff/drift
  - Drawback - when a new node joins, there is a lot of diff.

### Membership & Failure Detection
- Gossip based failure detection
- Explicit removal

#### Notes
- Focus on core of the problem
- Capacity planning only if needed
- Talk about internals e.g. DB internals
- Talk about Network protocols
- Trade offs
