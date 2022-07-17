# Trade Offs

## RDBMS vs Non-RDBMS (Key-Value store)

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
