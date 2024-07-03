# The Google File System (2003) 
GFS is a scalable distributed file system for large distributed data-intensive applications. It provides fault tolerance running on inexpensive commodity hardware, and it delivers high aggregate performance to large number of clients. 

## Motivation 
1. Component failures are norm, not exception
2. Large files
   *  multi-GB, TB 
   *  v.s. LFS: lots of small files
4. Workloads: append-only 
   *  e.x. log collections, web search
   *  large streaming reads, small random reads
   *  many large, sequential writes; none random writes

## Architecture 
![gfs-architecture](images/38-gfs/gfs-architecture.png)

* Control plane: single **master**
    *  Store metadata: namespace mapping
    *  Control: garbage collection, chunk migration, chunkserver states  
* Data plane: multiple **chunk servers**
    *  Store chunks on local disks as Linux files
      
## Fault tolerance & scalability: Replication 
**Chunks**: are replicated 3-way to handle faults
- Use large chunk size (64MB) 
    -  Handle lots of operations on a given chunks
    -  Reduce size of metadata stored on server and network communication in-between
    -  Beneficial for sequential writes
- Lease: maintain a consistent mutation order across replicas
- Chunk lease to one replica for each chunk (i.e. primary replica), which decides serial order for all mutations to the chunk
- Check data integrity using checksum blocks in each chunkserver
- Data forwarding is pipelined for efficiency, and is also network-aware

**Master**: replicated logs and checkpoints, has a shadow master 

## Relaxed consistency 
GFS deploys a relaxed consistency model: data is appended **_atomically_ _at least once_**. GFS assumes that client applications can handle the inconsistent state: i.e. filter out occasional padding and duplicate using checksums. This also helps improving performance. 

## Limitations 
Only one GFS master might become the single point of bottleneck for metadata operations. 
