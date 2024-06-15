# Design and Implementation of the Sun Network Filesystem (NFS) (1985) 
NFS is a **distributed file system** designed to make sharing of filesystem resources in a network of non-homogeneous machines easier. Design goals are 

1. **Crash recovery** (main goal): recover easily from server crashes
3. Transparency and UNIX semantics: access remote files as local 
4. **Reasonable performance**
5. Machine & os independence: being able to supply files to different types of clients 

## Techniques 
There are a few techniques used to achieve these goals 

### 1. Simple and fast recovery: _statelessness_
*  Stateless protocol: server does not keep track of anything about client
    *  most requests are **idempotent** (e.x. `LOOKUP`, `READ`, `WRITE`)   
*  Key structure: file handle (FD)
    *  <volume id, inode id, generation #>     
*  Steps 
    *  every client RCP call pass a FD
    *  server fails: client retires

### 2. Transparency: mounting
*  use `MOUNT` to attach a remote file system to a directory
*  Hostname lookup and address binding once per FS

### 3. Performance: caching 
#### 1) Client: R/W (cache consistency problem) 
   * _update visibility_: when do updates from client visible to others
       *  Sol: flush-on-close 
   * _stale cache_: server newer copy visible to other clients
       *  Sol: cache invalidation via `GETATTR` requests, with metadata cache

#### 2) Server: buffer writes 
* Performance v.s durability: if server crash
    *  put in battery-backed mem, or use faster medium to write
 
### 4. Machine & OS independences: VFS / Vnode interface 
- *Virtual File System (VFS)*: includes operations that are done to the entire file system
    - E.x. mounting, unmounting, getting FS-wide statistics, forcing dirty writes to disk, etc.
- *Virtual node (Vnode)*: includes all operations one can perform on a file
    - E.x. open, close, reads, writes, etc.
 
## Problem 
Security in early NFS implementation was lax: easy for any user on a client to masquerade as other users and gain access to virtually any file 
