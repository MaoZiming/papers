# Sun’s Network File System (NFS)


## A Basic Distributed File System 

- Client: client-side file system
    - A client application issues system calls to the client-side FS (i.e. `open()`, `read()`, `write()`, `close()`, `mkdir()`)
    - Transparent access to files
- Server: server-side file system (or the file server)
    - Reply to client request

## Intro

- One of the earliest and quite successful distributed systems
- Sun Microsystems developed an open protocol which specified the exact message formats that clients and servers use to communicate

![alt text](image-27.png)

Sun NFS instead developed an open protocol which simply specified the exact message formats that clients and servers would use to communicate.

## Focus: simple and fast server crash recovery 

Key to fast crash recovery: statelessness 

- Server does not keep track of anything about what is happening at each client
    - I.e. which clients are caching which blocks, which files are currently open at each client, file pointer position, etc.
- V.s. a stateful protocol
    - E.x. `open()` call from client side, return descriptor; the server fails and client issues subsequent read
    - File descriptor: **shared state** or **distributed state** between the client and the server
    - **Recover protocol**: client would make sure to keep enough information around to tell server what it needs to know
    - Get worse if the stateful server has to deal with client crashes (e.x. open and fail, when to close?)

- Now imagine that the client-side file system opens the file by sending a protocol message to the server saying “open the file ’foo’ and give me back a descriptor”. The file server then opens the file locally on its side and sends the descriptor back to the client. On subsequent reads, the client application uses that descriptor to call the read() system call; the client-side file system then passes the descriptor in a message to the file server, saying “read some bytes from the file that is referred to by the
descriptor I am passing you here”
- n this example, the file descriptor is a piece of shared state between
the client and the server (Ousterhout calls this distributed state [O91]).
### NFSv2 Protocol

- Crux: how to define the protocol to be both stateless and support POSIX file system API?
- File handle
    - *Volume identifier* —> which FS the request refers to
        - NFS server can export more than one FS
    - *Inode identifier* —> which file within that partition the request is accessing
    - *Generation number* —> needed when reusing an inode number
        - Increment it whenever an inode number is reused
        - Ensure that a client with an old file handle can’t accidentally access the newly allocated file

- `LOOKUP`
    - Used to obtain a file handle
    - Input: directory file handle and name of a file to look up
    - Output: file handle (or directory) plus its attribute
    - E.x. assume client already has file handle for root dir (`/`) (with **mount protocol**)
- `READ`
    - Input: file handle of the file, along with offset within the file and # of bytes to read
    - `WRITE` is handled similarly
- `GETATTR`
    - Given a file handle, it simply fetches the attributes of that file, including the last modified time
    - Important for caching

### From Protocol to Distributed File System 

- Client-side FS: tracks open files, and generally translate requests into set of protocol messages
- Server: simply responds

### Aside note

1. Idempotency is powerful! 
    - When an operation can be issued more than once, it is much easier to handle failure of the operation, you can just retry it
2. Perfect is the enemy of the good (Voltaire’s Law) 
    1. Corner cases: e.x. `MKDIR`
    2. Accepting life isn’t perfect and still building the system is a sign of good engineering 
3. Design for common case, and to make it work well


### Handling server failure with idempotent operations

- Server can fail to reply
    - 1) request lost
    - 2) server down
    - 3) reply lost on the way back from server
- Approach: client simply retries the request
- Property: most NFS requests are **idempotent**
    - **Def:** when the effect of performing the operation multiple times is equivalent to the effect of performing the operation a single time
    - E.x. `LOOKUP` and `READ`
    - E.x. `WRITE`
        - If, for example, a WRITE fails, the client can simply retry it. The WRITE message contains the data, the count, and (importantly) the exact offset to write the data to.
        - Thus, it can be repeated with the knowledge that the outcome of multiple writes is the same as the outcome of a single one
    - Some operations are hard to make idempotent
        - E.x. `MKDIR`


### Improving performance: client-side caching

- NFS client-side FS caches file data (and metadata) that it has read from the server in client memory
    - First access expensive, consecutive one fast
    - Cache: temporary buffer for writes
- **Cache consistency problem**
    - P1: update visibility - when do updates from one client become visible at other clients?
        - NFSv2 clients implement **flush-on-close** (a.k.a., **close-to-open**) consistency semantics
        - When a file is written to and subsequently closed by a client application, the client flushes all updates (i.e. dirty pages in the cache) to the server
        - This ensures that a subsequent open from another node will see the latest file version
        - Problem: temporary file soon being deleted
    - P2: stale cache - e.x. C1
        - NFSv2 client first check to see whether a file has changed before using its cached contents with `GETATTR` request
        - If out-dated, then client **invalidates** the file and removing it from client cache
        - Problem: flooded with `GETATTR` request
        - Remedy: attribute cache was added to each client; attribute for a particular file were placed in cache when file was first accessed, then would timeout
            - But hard to understand or reason about what exact version of file one was getting

### Implication on server-side write buffering

- If NFS servers may NOT return success on a WRITE protocol request until the write has been forced to stable storage
    - This might cause problem if the server crash
- If NFS commit each write to stable storage before informing success
    - Write performance can be bottleneck
    - Some tricks
        - First put writes in a battery-backed memory
        - Second is to use a FS specifically designed to write to disk quickly