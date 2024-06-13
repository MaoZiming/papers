# The Andrew File System (AFS)

## Introduction 

- How does it differ from NFS?
    - NFS is limited in scalability
        - E.x. frequent check if cache contents change
    - NFS cache consistency is hard to describe: depends on low-level implementation details
        - E.x. client-side cache timeout intervals
    - In AFS: when the file is opened, a client will generally receive the latest consistent copy from the server.
    - ![alt text](image-28.png)

## AFSv1

- Basic tenets of AFS: **whole-file cache** on **local disk** of client machine when accessing a file
    - Upon `close()`, the file (if modified) is flushed back to the server
    - V.s. NFS: caches blocks, and does so in client memory (not local disk)

- Details
    - Upon `open()` call
        - Client sends a *Fetch* protocol message to server with entire pathname (e.x. `/home/remzi/notes.txt`)
        - Server traverses the pathname, finds the desired file, and ships the file back to client
        - Client caches the file on local disk
            - Consecutive R/W: also uses local memory to cache blocks on its local disk
        - When finish
            - If modified, flush new version back to the server with *Store* protocol message

- Next-time file accessing
    - Client first contacts the server (with *TestAuth* protocol message)
        - Determine whether the file has changed
        - If not, use the local cache copy
    - Early version: only cache file contents; directories were only kept at the server

## Problem with V1

Scalability of AFS is largely limited: the server CPU became the bottleneck of the system, and each server could only service 20 clients. 

There are two main problems with AFSv1:

1. **Path-traversal costs are too high**
    1. *Fetch* / *Store* request: pass entire pathname 
    2. CPU time simply walking down directory paths when there’re too many clients 
    3. Server spends too much time traversing pathnames
2. **The client issues too many TestAuth protocol messages** 
    1. Just like NFS problem 
3. Additional problem 
    1. **Load imbalance** across servers 
        1. Sol: volumes —> an administrator can move across servers to balance load 
    2. Server used a single distinct process per **client** thus inducing **context switching** and other overheads 
        1. Sol: building the server with threads instead of processes

## AFSv2

### #1: Callback

- AFSv2 introduce **callback** to reduce # of client / server interactions, adding this **state** to the system
- Client assumes that the file is valid until server tells it
- Analogy to: pulling v.s interrupts

### #2: FID

- Similar to NFS file handle; consists of
    - Volume identifier
    - File identifier
    - “Uniquifier”: to enable reuse of the volume and file IDs when a file is deleted
- FID replaces pathnames to specify which file a client was interested in
    - Client walk the pathname, caching the results
    - Example
- Key difference from NFS
    - With each fetch, AFS client would establish a callback with the server
    - Benefit: second access as fast as accessing the file locally (no server interaction)
  
- For example, if a client accessed the file /home/remzi/notes.txt, and home was the AFS directory mounted onto / (i.e., / was the local root directory, but home and its children were in AFS), the client would first Fetch the directory contents of home, put them in the local-disk cache, and set up a callback on home. Then, the client would Fetch the directory thus ensuring that the server would notify the client of a change in its cached state. 
- 


### Cache Consistency 

Two important cases:

1. Consistency between processes on *different* machine 
    1. AFS makes updates visible at the server and invalidates cached copies when the updated file is closed 
    2. Server then “breaks” callbacks for any clients with cached copies
    3. Concurrent write: **last writer wins** to the entire file 
        1. V.s. NFS: final file could end up as mix of updates from clients 
2. Consistency between processes on the *same* machine 
    1. Write immediately visible to other local processes (i.e. not wait until file closed) 
    2. Based upon typical UNIX semantics

### Crash Recovery 

- More involved with NFS, where clients hardly noticed a server crash
- Scenario: S cannot content some C1 for a while, but send the callback messages
- Scenario: S crashes, and callbacks are kept in memory
- Some approach
    - Server send a message when it is up and running again
    - Clients check server is alive periodically (via heartbeat)

- Observation
    - In many cases, performance of each system is roughly the same
        - Note: writing to disk buffered, reading from disk cached
    - Interesting difference: large-file sequential re-read
        - Workload 6
        - Read from disk cache v.s read remote
        - NFS also increase server load, limited in scale
    - Sequential writes should perform similarly
        - Workload 8, 9
    - AFS performs worse on sequential file overwrites
        - Workload 10
        - Overwrite for AFS: first fetches the old file entirely, subsequently overwrite
        - Overwrite for NFS: simply overwrite blocks and avoid initial useless read
    - Workloads that access a small subset of data within large files perform much better on NFS than AFS
        - Workloads 7, 11
        - NFS, as a block-based protocol, performs I/O that is proportional to size of R/W

## Other Improvements on AFS

- True global namespace to clients: all files were named the same way on all clients
    - V.s. NFS: allow each client to mount NFS server in any way
- Security mechanisms to authenticate users and ensure files private if user desired
    - V.s. NFS: primitive support for security
- Flexible user-managed access control (e.x. who exactly can access which files)
    - V.s. NFS: much less support for this type of sharing
- Tools to enable simpler management of servers for system admins

Sadly, NFS dominates the market place and becomes an open standard. NFSv4 now also adds server state (e.g. an “open” protocol message).