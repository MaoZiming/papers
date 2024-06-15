# Disconnected Operation in the Coda File System (1991)  
This paper presents Coda File System, based AFS. It was designed for mobile clients that disconnect as their machines move. 

To make disconnected operation transparent, each client keep a cached copy of remote files once connecting to the server. When disconnected, clients can work on the local cached copy without accessing to server. Once connected back to the server, clients synchronize updated contents with the server and download new files to its cache. 

## Key Motivation: Availability 
Coda wants to support disconnected operation, i.e. enables a client to continue accessing critical data during temporary failures. Availability is achieved by replication and disconnected operation with cache
-   **Server replication** for performance, scalability, and availability
    -  consistency ensured by callbacks  
-   **Client replication** for (more) availability
    -  client continues to work from local cache when disconnected 
-   **Optimistic replica control** to make it all work
    -  allow copies to diverge, detect, and resolve conflicts 

## Disconnected operations: state transition 
* **Hoarding**: pre-caching 
  * client is connected and actively downloaded files from server and keep a cache locally 
  * balance current working set v.s. future needs  
  * hoard walk periodically restores equillibrium between recent and explicit cache
  * refetch purged cache object upon next hoard walk
  * directories allowed to become inconsistent as most operations are adding or removing items
* **Emulation**: psuedo-server 
  * replace the function of the server
  * logging for fault-tolerance
    * save replay log locally
    * compact entries to log to limit cache growth
  * RVM (camelot) keeps integrity of metadata during disconnected operation
* **Reintegration**
  * propagates changes back to servers (i.e. transfer updates, resolves conflicts) 
  * on conflicts, users use a debugging program to replay selectively
    
## Optimistic v.s pessmistic replica control 
1. Pessimistic 
    1. Only one partition is allowed to access files during network partition
    2. Pros: always consistent
    3. Cons: unavailability
        1. Difficult to handle involuntary disconnection (i.e. if it held a lock) 
        2. Errant client can block other client indefinitely
2. Optimistic 
    1. All partitions can access files 
    2. Conflicts are detected and resolved later
    3. Pros: high availability
    4. Cons: conflicting writes, relies on resolution protocol  
    5. Rational of using this: chance of conflicts is low because **low degree of write share in UNIX**
