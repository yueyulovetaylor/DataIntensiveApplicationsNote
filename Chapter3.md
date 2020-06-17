# Chapter 3: Storage and Retrieval

General Idea of this chapter: How we can store the data we are given, and how we can find it again when we are asked for it.

## 1. Data Structures That Power Your Database
The simplest database with `db_get()` and `db_set()` functions in bash.
```bash
#!/bin/bash
db_set () { echo "$1,$2" >> database }
db_get () { grep "^$1," database | sed -e "s/^$1,//" | tail -n 1 }
```
* Similar to the simplicity of `db_set()`, many databases internally user a log, which is an append-only data file.
* The cost of db lookup (`db_get()`) is O(n), so we want to introduce index in this chapter, acting as a signpost and help you to locate the data.
* An important trade-off: **Well chosen indexes speed up read queries, but every index slows down writes**

### 1.1 Hash Indexes
* Simplest Indexing Strategy: **keep an in-memory hash map where every key is mapped to a byte offset in the data file**. Please refer to illustration below.\
  <img src="./Images/Chapter3/HashIndex.png" height=60% width=60%>
* An example of this model is Bitcask and the situation is where the value for each key is updated frequently.
* How do we avoid eventually running out of disk space? Please pay attention to the following points with regarding to this issue. 
  * General Strategy: Break the log into segments of a certain size by closing a segment file when it reaches a certain size, and make subsequent writes to a new segment file.
  * Compaction: Throw away duplicate keys in the log, and keep only the most recent update for each key.
  * We can also merge several segments at the same time as performing compaction. Merging and compaction can be done in a background thread. 
    * We can still read into the old segment files and write request to the latest segment file.
    * After merging and compaction is complete, we swtich the read request to the new merged segment and the old segment files can be deleted.
  * Please refer to illustration below for how compaction and merge works.\
    <img src="./Images/Chapter3/CompactionAndMerge.png" height=60% width=60%>
  * Each segment keeps its own hash table, mapping keys to file offsets.
* Implementing Details:
  * File format: Use a binary format that first encodes the length of the string followed by the raw string.
  * Delete Record: Append a special delete record to the datafile (**tombstone**), when deleting, discard all previous values for the deleted key.
  * Crash Recovery: Store a snapshot of each segment's hash map on disk, which can be loaded to memory more quickly.
  * Partially Written Record: Include checksum so that corrupted parts of the log can be detected and ignored.
  * Concurrency Control: Only one write thread, but allow multiple concurrent read threads.
* Benefit of append-only design:
  * Appending and segment merging are sequential write operations, which are much faster than random writes.
  * Concurrency and crash recovery are mush simpler.
* Limitations of hash table index:
  * Too many hash keys will be out of memory.
  * Range queries are not efficient.

### 1.2 SSTables and LSM-Trees
* From now on, we begin to require the sequence of key-value pairs to be **sorted by key**. We define this data structure as **Sorted String Table (SSTable)**.
* Advantage of using SSTables:
  * Merging segments is simple and efficient, since the strategy behind this is very similar to mergesort algorithm.
  * Do not need to keep an index of all the keys in memory. Just need some indexes which can be sparse.
  * We can group and compress records into a block before writing it to disk, each entry of the sparse in-memory index points at the start of a compressed block.\
    <img src="./Images/Chapter3/SSTable.png" height=60% width=60%>

#### 1.2.1 Constructing and Maintaining SSTables
  * When a write request comes, add it to a in-memory balanced tree structure (red-black tree or AVL tree), which serves as a memtable.
  * If a memtable reaches a threshold, store the memtable into the newest segment file of the database. Write request can continue to a new memtable instance.
  * When read request comes, it will start from the in-memory memtable to most recent on-disk segment file, then in the next order segment, etc.
  * A background thread will keep running a merging and compaction process.

#### 1.2.2 Making an LSM-Tree out of SSTables
  * Storage engines that are based on merging and compacting sorted files are often called LSM storage engines.

#### 1.2.3 Performance Optimizations
  * Looking up keys that do not exist in the database can be slow. So additional Bloom Filter is usually implemented in order to tell whether a key exists in the database or not.
  * Basic idea of LSM-Tree is to **keep a cascade of SSTables that are merged in the background**. Two different compaction strategies:
    * Size-tiered compaction: newer and smaller SSTables are merged into older and larger ones.
    * Leveled compaction: key range is split up into smaller SSTables and older data is moved to "separate" levels. This allows compaction to be more incremental and use small disk size.

### 1.3 B-Trees

### 1.4 Comparing B-Trees and LSM-Trees

### 1.5 Other Indexing Structures

## 2. Transaction Processing or Analytics

## 3. Column-Oriented Storage