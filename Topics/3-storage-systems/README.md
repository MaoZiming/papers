 # Storage Systems 
| Local FS  | Distributed FS |
| ------------- | ------------- |
| [FFS](https://github.com/lynnliu030/os-prelim/blob/main/storage/ffs.md) (1984) | [NFS](https://github.com/lynnliu030/os-prelim/blob/main/storage/nfs.md) (1985) |
| [JFS](https://github.com/lynnliu030/os-prelim/blob/main/storage/crash_consistency_jfs.md) (1990) | [AFS](https://github.com/lynnliu030/os-prelim/blob/main/storage/afs.md) (1988) |
|  [LFS](https://github.com/lynnliu030/os-prelim/blob/main/storage/lfs.md) (1991) | [CODA](https://github.com/lynnliu030/os-prelim/blob/main/storage/coda.md) (1991) | 
|  | [Sprite](https://github.com/lynnliu030/os-prelim/blob/main/cluster_computing/sprite.md) (1991) |
|  | [GFS](https://github.com/lynnliu030/os-prelim/blob/main/storage/gfs.md) (2003)|

Other paper:
* [Rethink the Sync](https://github.com/lynnliu030/os-prelim/blob/main/storage/rethink_sync.md)
* [Chord](https://github.com/lynnliu030/os-prelim/blob/main/storage/chord.md)
  
## Objectives: performance, scalability, durability, availability
* FFS optimizes performance by changing the on-disk FS layouts
* JFS optimizes for crash consistency by using journalling and re-do logging
* LFS optimizes for small-write performance by write buffering
* NFS optimizes distributed FS crash-recovery by deploying a stateless protocol
* AFS optimizes distributed FS performance by using the callback mechanism, putting some states in the server
* CODA optimizes for availability when the client is disconnected, by using server and client replication, and optimisitic replica control for conflict resolution
* GFS is modern distributed file system that optimizes in DC environment where there is large file, append-only workloads and faults are normal. It deploys a centralized scheduling scheme, decouple control and data (use large chunks), uses data replication to improve scalability and fault tolerance.
* External synchrony balances the durability v.s performance trade-offs of synchronous and asynchronous I/O via a user-oriented approach
* Chord optimizes for scalable information lookup in large-scale p2p distributed storage system 

## Small v.s large files 
There is a design trade-off of FS depending on whether they are optimized for large number of small files, or few number of large files. Decisions like block size, number of inodes, batching etc affect this. 

### Small files 
* Pros: high concurrency, lower latency, easier to distribute across multiple nodes
* Cons: increase metadata overhead, potential file count limitations

### Large files
* Pros: reduced metadata, easier management, efficient for sequential access
* Cons: not optimized for high concurrency, potential for data corruption affecting the whole file, internal fragmentation 

For example, GFS chooses larger block size to reduce the metadata size and improve sequential access performance. Traditional FS uses a much smaller block size. 

## Synchronous v.s Asynchronous I/O 
**Synchronous I/O**: application blocks until the I/O completes
- Pros: durability and reliability
- Cons: slow 

**Asynchronous I/O**: enable application to issue an I/O request and return control immediately to the caller, before the I/O has completed
* **Poll**: application poll the system via system call to determine whether I/O has completed
* **Interrupt**: UNIX signals to inform applications when async I/O completes
- Pros: performance
- Cons: durability and reliability 
  
## Sequential v.s random reads / writes 
### Sequential access 
* Pros: more efficient disk I/O, better use of cache
### Random access 
* Cons: less efficient disk utilizations

LFS is optimized for write-heavy workloads by accumulating small random writes and flushes them to disk sequentially. This approach minimizes disk seek time and makes efficient use of disk bandwidth. GFS is designed to handle a few large files and focuses on optimizing sequential reads and appends (i.e. batch processing workloads like MapReduce). 

## Trade-offs 
### 1. Performance v.s durability (crash recovery) 
File systems that prioritize performance might use strategies like delayed writes, write caching, or large write buffers to improve speed. However, these decrease the durability. And if the system crashes before the data is persistently stored, affecting the system's ability to recover from crashes.

Examples like JFS uses journalling and redo-logging to ensure crash consistency and faster recovery, but can incur a performance cost on every write.

In distributed FS, NFS uses a stateless protocol for faster crash recovery, but can run into scalability issue because the need of repeated cache validation between client and server. AFS adds some states to the server, but makes crash recovery more complicated. 

External synchorny balances performance and durability trade-offs by providing guarantees directly to the users. 

### 2. Consistency v.s availability 
In distributed file systems, maintaining strict consistency often comes at the cost of availability. If a node fails or network partitions occur, strictly consistent systems may become unavailable until the issue is resolved.

CODA: Optimizes for availability when the client is disconnected, using server and client replication and optimistic replica control for conflict resolution. However, this can potentially compromise strict consistency (also a trade-off of pessimistic and optimistic concurrency control). 

### 3. Consistency v.s performance 
In many systems, the more consistent you make your system, the more you might have to sacrifice in performance because ensuring consistency often involves additional communications between nodes, which can slow down operations.

GFS deploys a weaker consistency model but allows the system to perform great at scale. 
