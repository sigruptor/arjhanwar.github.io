## Storage and Retrieval
Application developer need to have some idea about how storage engine manages the data, this helps them select a storage engine that would work best for their application.

### Data Structures that power your Database
Starting the model with key-value pairing. Writes are very simple
- Keep appending to a file
- However, reads becomes very expensive O(n).
- Maintain additional metadata (indexes) (key -> offset where data is stored).
a) However, this adds a little more complexity to the writes as you to have to write at one additional place.
b) Reads becomes simple, quick lookup allows you to read in O(1).
- Speeds up reads but slows down write, a trade-off that application has to sign up for.

#### Hash Indexes
Keep an in-memory hash-map of the keys with the offset of bytes in the file (location where value can be found).
If dataset is larger than RAM size, it can be written on to disk as well. If part of data is in-memory it does not require disk seek for read.

- If there are frequent updates to a less number of keys (fitting in memory), this works great.
**How to avoid eventually running out of disk space ?**
When a file reaches a certain size, subsequent write goes to a new segment and a Compaction is performed on the segments. Merging of segments happen in 
background.

To read: 
1. First check memory hash table
2. Then segment in order (new ones to old ones), older one would have stale data if the keys are written again.

Issues:
1. File Format: CSV is not best format for log, simpler to use binary format that encodes the length of a string in bytes following by raw string.
2. Deletion: Need to add a tombstone record for deleted ones, which tells system to discard all older values.
3. Crash Recovery: Read the segments to load in memory hash table, but at times this could be slow. Some systems keep a snapshot of each segments hash map.
4. Partially written records: If crash happens halfway through writes, system must have checksums, allowing corrupted parts of log to be detected and ignored.
5. Concurrency: One writer thread, data file segments are append only, so they can be read by concurrently multiple threads.

Append only log :
- Appending and merging segment trees are sequential write operations, which are generally faster than random writes on magnetic disks (even on SSDs)
- Concurrency & Crash Recovery are simpler if segment files are append only or immutable. Dont have to worry about modifying the data (being halfwritten etc.)
- Merging of segment avoids the problem of fragmentation and duplication.

##### Limitations
- If data does not fit in memory, hash map though can be maintained on disk but that is usually hard
- Range queries are not efficient, cant scan over the keys between abc...def.

#### SSTables & LSM-Trees
##### SS Tables - Sorted Set Tables
Earlier we have had much of log-structured segments - that was a sequence of key value pairs in the order they were written. Though in SSTables
Segment of key-value pairs is sorted by keys. 

1.  Managing segments is simple, if files becomes large than a particular size its written to disk segment
2.  Compaction is similar to merge sort, each of the segment spits out the different keys and than a merge process converges those keys into a new segment.
3.  Since after the merge new segment is written, it still follows sequential write.
4.  In-memory index could really be sparse (it does not need to keep all the keys in memory), since it is sorted. It can be figured from prev and next set 
of keys.
5. Since read requests need to scan over a several key-value pairs in requested range, its possible to group these records into a block and compress it
before writing it to disk. Each entry of spark-in-memory index then points at start of compressed block. Besides saving disk space, compression also reduces the I/O bandwidth use.

###### Constructing and maintaining SSTable
- When write comes in, add it to in-memory balanced tree data structure (memtable)
- When memtable gets bigger, write it to disk as an SSTable...
- A WAL is used so that if a crash happens before file is written
- Merging and compaction process can be run in background
- For reads, first read from memory, then segments (in order they are written).

###### LSM Trees
Storage engines based on the principle of Log-structured filesystems and which follows principle of merging and compacting sorted files are often called 
LST storage engines.

Optimizations
1. Bloom filters - helps the system decide if a key does not exist.
2. Different strategies to determine order and timing of how SSTables are merged and compacted.
3. Since data is sorted range queries are optimal.

#### BTrees
Similar to SSTables, keys are sorted. BTrees break the database into fixed size block pages (4KB) similar to disk pages. Each page is identified by a location

- There a root node, and each level has a range of keys. Branching factor determines the split. The page contains several keys and references to child pages.
- Actual location of record is stored on the leaf node
- For updating a key, we need to find the page of the key, if there isnot any space it is split into two half ffull pages, parent page is updated to account for new revision.

Reliable:
- WAL 
- File is updated in place this is on-contrast with LSM.
- Overwrititng the page on disk can be considered as an actual hardware operation. Magnetic disk, SSDs (must erase and write large block).
- However some operation require several different pages to be overwritten.
- FOr updating pages, some latches (light weigth locking) 

Optimizations:
- Not all keys need to be stored
- Keys on the leaf node are kept sequentially this helps with a) Range queries b) Data is usually requested in a batch.
- Instead of overwriting, a new page is created (snapshot isolation)

#### Comparison BTrees vs LSM Trees
LSM Trees are said to be faster for writes, while BTrees are said to be faster for reads (however this depends on the workload).

- THey have less write amplification factor (for SSDs, even overwriting requires entire block to be re-written, and SSD have fixed number of writes).
- FOr compaction, the Segments written are sequential (faster than random writes on magnetic disks as well)
- BTrees usually leave some fragmented space
- Compaction process can sometime interfere with performance of ongoing reads and writes. 
- SSTables does not throttle the write requests
- BTrees key exists exactly at one place in the index - useful for databases where transactional semantics are needed.

#### Other Indexing Structures
For relational databases, BTrees, SSTables can be used as secondary indexes. 
1. Clustered Index: Value is stored in the index itself, reads are much faster this way.
2. NonClustered Index: Value is not stored in the index, rather the pointer to the row is stored. This way row need not be duplicated during writes, 
however during reads, there is one extra hop

###### Multi-Column Index
Index with multiple columns, designed to serve very specific queries.
- Location indexes (R Index) : usually 2 dimensional location is converted into a single number.

###### Full Text/Fuzzy indexes
Where the word that is searched is not exactly the same, the goal is to retrieve similar words or synonyms. **Lucene** is able to search the words upto
a certain edit distance. Usually SSTAbles - sparse inmemory index with a collection of keys. However, in Lucene, in-memory index is a finite state automatan over the characters of a key (Trie)

###### Keeping everything in memory
This is a faster solution as system does not have to figure out the encoding-in memory data structure into disk. This can be made to work with WAL and 
recovering from logs when the system crashes.
**Redis allows you to store SOrted Sets or priority queues directly as in-memory implementation is simple.**
 

### Transactional Processing & Analytics
OLTP - online Transaction Processing, Bank transctions, game etc..Applications are mostly interactive, user queries a bunch of records, read/update them.
OLAP - Online Analytical Processing - databses being used for analytics. These queries instead of returning raw data would read entire database and return
aggregates (sum, avg etc.)

| Property      | Transactional Processing System OLTP  | Analytic Systems (OLAP) |
| ----------- | ----------- | ----------- |
| Main Read Pattern      | Small number of records per query, fetched by keys       |   Aggregate over large number of records          |      
| Main Write Pattern   | Random-access, low-latency writes from user input        | Bulk import (ETL) or event stream |
| Primarily Used By   | End user/customer via web app        | Internal analyst for decision support | 
| What data represents   | latest state (current point in time)        | history of events that happened over time | 
| Dataset Size   | GBs to TBs        | TBs to PBs | 

### Data Warehousing
Databases for analytical processing are called datawarehouse. OLTP systems are to be highly available, low latency systems, often critical to the business. Doing analytical processing on these potentially impacts the performance & experience of the user. So having separate OLAP is useful.

Ecommerce -> Service (Sales DB) -> Extract -> transform -> load - data warehouse
Stock app -> Stock Service (inventory DB) -> Extract -> transform -> load - data warehouse

Having a separate DB allows for optimization of system for analytical queries.

#### Column Oriented Storage
Relational database stores data in a row i.e. entire row is stored on the DB together. Useful when entire row is needed for queries (useful in Business use cases). However for analytical purposes, only certain columns are needed to be fetched together. With relational model, query needs to load everything in memory, filter and return the result. e.g. `Select facts, week from Table`, queries like `Select * from table` are not common for analytical purposes.

Idea is to not store entire row together, but store values from each column in the same file. e.g table with 5 columns (product, customer, price...), it will be stored in 5 files for one column each. Each file would store the same row at the same line number e..g 5th row of each file would represent the detail about the same transaction/entity. 
Storing in a column oriented manner allows:
1. query efficiently for analytical purposes, where you only need to read specific files.
2. You could enable column compression (Bitmap encoding - you dont need to store all the values, e.g. billion rows might have fixed set of values around 200, it could be encoded in bitmap. Moreover Runtime encoding on it can add to optimizations further.
3. This allows for very efficient filtering (you could just AND/OR the results of bitmaps)
4. e.g. Parquet, cassandra

##### Sort Order
Data can be sorted based on column values, this can add to more optimizations (compression)
- You could have different sort orders.
- However writing can be combersome, but we can combine this with in-memory LSM/BTrees.

### Summary
There are 2 types of storage engines.
1. OLTP (Online Transactional processing): 
- User facing, they see a huge volume of request
- Only a few set of records are relavant for each query.
- Application records query based on a key, and an index is usually used to answer that query.
- Data seek time is often the bottleneck here.
2. OLAP (Online Analytical Processing):
- Data warehouse, used by analytics and not by end users.
- VOlume of query is not as much but each query involves reading millions of rows/columns in a short time.
- Disk bandwidth (not seek time) is often the bottleneck here, column oriented storage is a proper solution for these workloads.

OLTP has 2 schools of engines
1. Log Structured: aPpend only system, never updates a file that is already written,
- LSM trees, Cassandra, Hbase, Lucene are examples
2. Update-in place: treats disk as a set of fixed size pages that can be overwritten
- BTrees used in all major relational databases.

When queries require sequentially scanning cross a large number of rows, indexes are much less relevant. It becomes important to encode the data and minimze the amount of data query needs to read from disk.
 
#### Considerations
1) Does data fit into memory ? 
2) Is load read-heavy or write heavy ?
3) Are the updates more frequent to a particular set of keys ?
4) Handling the writes in sequential order ?
5) If in-memory index is enough, would provide a lot of advantages (storing priority queue directly)


