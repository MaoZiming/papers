# Scale and Performance in a Distributed File System (AFS) (1988)  
> Probably the earliest clear thinking on how to build distributed file systems. A wonderful combination of the science of measurement and principled engineering.

This paper discusses scale and performance in Andrew File System (AFS) developed at CMU, and the changes made to the original prototype to address these issues. The file system was motivated primarily by considerations of scale. Large scale affects distributed system in terms of performance and operation cost. 

## Motivation: scale 
Key problems of original AFS prototype via extensive benchmarking: 
1. **Path-traversal** costs are too high
    1. *Fetch* / *Store* request: pass entire pathname 
    2. CPU time simply walking down directory paths when there’re too many clients 
2. The client issues **too many TestAuth** protocol messages 
    1. Cache validation (similar to NFS) 
3. Excessive **context switching** overhead and high virtual memory paging demands 
4. Server **imbalance**

## Techniques

### Scale 
**Cache mgmt (callback interface)**: rather than checking with server on each open, AFS client instead cache entries of directories and symbolic links, and assume the cache entries are valid unless notified (_callback_). This reduce the cache validation traffic and reduce loads on server. 
* Stateful: server keeps track of states, which make crash recovery harder!
  
**Name resolution**: move the name resolution from server to client, client walks and maps the pathname to Fid (i.e. <vol #, vnode #, uniquifier>. 

**Server process**: a server process per client did not scale well; instead use a single process to service all clients, use threads instead of processes. 

### Usability 
Storage representation: instead of pathname, access files by their inode. 

AFS usability changes: 
* true global namespace to clients and ensure all files were named the same way on all client machines (v.s. NFS)
* considers security and incorporates mechanisms to authenticate users as well as providing user-managed access control
* simpler management of servers. 

## Crash recovery 
Crash recovery is more involved than NFS with the callback mechanism (e.x. may miss callback recall messages during crash). When client recovers, it needs to check with server again. 

Server recovery is also harder, as callbacks are kept in memory. Methods to deal with it can be having server sends message ("don't trust cache!") or client checks with server periodically with heartbeat. NFS server recovery is immediate as it doesn't keep any states. 

## Performance 
**Whole-file caching in AFS**: good when large files are frequently re-read, as it can serve the files directly from the local disk cache and reduce server load and network latency. 
* files reads usually went to the local disk cache (and potentially local memory) 

**Block-level caching in NFS**: better in use-cases where only a small subset of a large file is read or modified.
* NFS can fetch or update only the necessary portions of a file, making it more efficient for workloads involving random or partial file access.
* Also overwriting existing files is better in NFS, as it avoids the initial read operation that AFS requires to fetch the whole file before overwriting.
