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

#### Comparison BTrees vs LSM Trees

#### Other Indexing Structures

#### Considerations
1) Does data fit into memory ? 
2) Is load read-heavy or write heavy ?
3) Are the updates more frequent to a particular set of keys ?
4) Handling the writes in sequential order ?



