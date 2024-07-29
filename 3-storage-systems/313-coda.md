# Disconnected Operation in the Coda File System (1991)  

Link: [Coda File System](https://www.cs.cmu.edu/afs/cs/project/coda-www/ResearchWebPages/docdir/s13.pdf)

Read June 25th, 2024.

* This paper presents Coda File System, based on AFS. It was designed for mobile clients that disconnect as their machines move. 

* To make disconnected operation transparent, each client keep a cached copy of remote files once connecting to the server. When disconnected, clients can work on the local cached copy without accessing to server. Once connected back to the server, clients synchronize updated contents with the server and download new files to its cache. 

> It is specifically not intended for applications that exhibit highly concurrent, fine granularity data access.

## Key Motivation: Availability 
* Coda wants to support **disconnected operation**, i.e. enables a client to continue accessing critical data during temporary failures. Availability is achieved by replication and disconnected operation with cache

* Clients view Coda as a single, location-transparent shared Unix file system. The Coda namespace is mapped to individual file servers at the granularity of subtrees called volumes. 
* At each client, a cache manager (Venus) dynamically obtains and caches volume mappings.
* The set of replication sites for a volume is its volume storage group (VSG). The subset of a VSG that is currently **accessible** is a client’s accessible VSG (AVSG).

* While disconnected, Venus services file system requests by relying solely on the contents of its cache. **Cache misses are appeared as failures.**

-   **Server replication** for performance, scalability, and availability
    -  consistency ensured by **callbacks**  
 -  First-class replicas (servers) coupled with second-class replicas (clients). With disconnection, the quality of the second-class replica will degrade.
-   Client continues to work from local cache when disconnected 
-   **Optimistic replica control** to make it all work
    -  allow copies to diverge, detect, and resolve conflicts 
 -  whereas **pessimistic replica control** requires exclusive accesses. 
    -  A disconnected client with shared control of an object would force the rest of the system to defer all updates until it reconnected. 

* For scalability, it uses **callback-based cache coherence**, and **whole-file caching** (similar to AFS); as well as placing of functionalities on clients.
  * For whole-file caching, a cache miss can only occur on an open, never on a read, write, seek, or close. This simplifies the implementatation of disconnected operations. 
  * Partial-file caching scheme would have complicated the implementation. 

### Design Goal

Availability is the major concern

- Want to support **disconnected operation**
- i.e. enables a client to continue accessing critical data during temporary failures
- **Key idea:** availability by replication and disconnected operation with cache

### Design Rationale

- **Replicate across servers** for performance, scalability, and availability
    - Servers have higher quality (space, security, maintenance)
- Use **optimistic replica control** to make it all work
    - Locks and leases are too limiting
        - Locks: reserve resources for too long
        - Leases: reserve resources for not long enough
          - Placing a time bound on exclusive or shared control, as done in the case of leases [7], avoids this problem but introduces others. Once a lease expires, a disconnected client loses the ability to access a cached object, even if no one else in the system is interested in it. This, in turn, defeats the purpose of disconnected operation which is to provide high availability. Worse, updates already made while disconnected have to be discarded.
    - Optimistic control may result in (W/W) conflict
        -**Need conflict detection and resolution**, but uncommon for UNIX env
    - UNIX write sharing is uncommon. Consistent with the goal of maximizing availability.

### Architecture

- ***HOARDING:*** Pre-caching for disconnected information access
    - Client is connected and actively downloaded files from server and keep a cache locally
    - Balance current working set v.s. future needs
        - Hoarding DB (HDB) provides user-specified list of files
            - Prioritized, may include directories and their descendants
            - Hoard profiles (files and their priorities)
        - Cache organized **hierarchically**
            - I.e. ancestors of a cached object must be cached
            - > To resolve the pathname of a cached object while disconnected, it is imperative that all the ancestors of the object also be cached. Venus must therefore ensure that a cached directory is not purged before any of its descendants. 
            - Assign infinite priorities to directories with cached children. 
        - Hoard walking: maintains equilibrium in the cache. (every 10 minutes in the implementation)
            - Goal: no uncached object has higher priority than cached ones
            - > Once an object is modified, it is likely to be modified many more times by the same user within a short interval. 
            - Operation
                - Phase 1: Re-evaluate name bindings to identify all children (i.e. any new children created by other clients?)
                - Phase 2: Re-calculate priorities in cache and per-workstation hoard database (HDB), evict and fetch as needed
    - Callback breaks during Hoarding
        - Purge files, symlinks immediately
        - Refetch on-demand or during the next hoard walk.
        - Delay directory operations (do not purge directory)
          - The reason is that a callback break on a directory typically means that an entry has been added or deleted from the directory. Other directory entries are likely fine. 

![alt text](images/313-coda/hoarding-state-machine.png)

- ***EMULATION:** Psuedo-server*
    - Client disconnects but acts like it is connected by working on the cached contents
        - **Modified objects assume infinite priority**
          - So they are not purged before reintegration.
        - Log operations for later REINTEGRATION
            - Replay log (metadata, HDB) accessed through recoverable virtual memory (RVM)
              - > In case of a crash, the amount of data lost should be no greater than if the same failure occurred during connected operations. 
            - Contains store operations, but not individual writes
            - Removes previous store records on repeated close
            - Should remove stores after unlink or truncate
            - Should remove log records that are inverted
            - Cancellation of old log records (e.g. `rmdir`)
- ***REINTEGRATION***
    - **Perform a volume at a time, where the operations to a volume is suspended.**
    - Propagates changes back to servers (i.e. transfer updates, resolves conflicts)
        - Changes at one end: updates
        - Changes in both ends: W/W conflict
    - Ship log to all servers in AVSG
        - Begin transaction, lock all referenced objects
        - Validate each operation and then execute
        - Perform data transfer (back-fetching)
        - Commit transaction, unlock all objects
        - On error, abort and create replay file
  
## Disconnected operations: state transition 
* **Hoarding**: **pre-caching**
  * client is connected and actively downloaded files from server and keep a cache locally 
  * balance current working set v.s. future needs  
  * hoard walk periodically restores equillibrium between recent and explicit cache
    * Some cached file prioirty is lower than the uncached files' priority.
  * refetch purged cache object upon next hoard walk
  * directories allowed to become inconsistent as most operations are adding or removing items
  * Implicit information (recent reference history); Explicit information (per-work station user profiles.)
  * Hoard priority
  
* **Emulation**: psuedo-server 
  * replace the function of the server
  * logging for fault-tolerance
    * save replay log locally
    * compact entries to log to limit cache growth
  * RVM (camelot) keeps integrity of metadata during disconnected operation
* **Reintegration**
  * propagates changes back to servers (i.e. transfer updates, resolves conflicts) 
  * on conflicts, users use a debugging program to replay selectively
  * In the case of a *store* of a file, the entire re-integration is aborted.
  * In the case of a *store* of a directory, a conflict is declared only if a newly created name collides with an existing name, etc.
  * Marking conflicting replicas inconsistent and forcing manual repairs.
    
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

## Adoptions

* Disconnected session of one hour requires a minute for reintegration
* A local disk capacity of 100MB on our clients have been adequate for our initial session.

> Coda is unique in that it exploits caching for both performance and high availability while preserving a high degree of transparency. 

> View [Coda] as an idea of write-back caching to mask temporary failures.