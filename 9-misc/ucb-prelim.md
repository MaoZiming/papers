# UCB Prelim Questions

Link: https://www2.eecs.berkeley.edu/Protected/Grads/CS/Prelims/osqu.html

## (Fall 2015 - Stoica & Brewer):

### Question 1: Non-volatile fast storage

Recently IBM and Micron have introduced a revolutionary storage technology, called 3D Xpoint, which promises non-volatility as well as a performance close to RAM's performance, i.e., just 2.5x higher access latency and 50% lower memory bandwidth than today's DRAM. Additionally, 3D XPoint can provide up to 10x higher density than DRAM.

Assume you are asked to develop a new OS for a machine that uses exclusively 3D XPoint technology, i.e., no disk drives, and no SSDs. 


#### New stab of the answer

* Simplify read and write buffer.
* Simplify I/O operations (byte addressable, access are at cacheline granularity just like the main DRAM rather tha block addressable). No block layer. 
  * Writes to PM are cache-line level. 
* Write-ahead logging is not needed.
* Unified memory and storage -> Unified address space.
* In-memory data structure with persistence. 

#### Questions

1) Discuss how you can simplify the design of your OS? Think about files system, virtual memory, etc.

* The OS can treat 3D XPoint as a single large pool of memory, eliminating the need for separate memory management for RAM and storage management for disks/SSDs. Files can be accessed directly from 3D XPoint, reducing the need for extensive caching mechanisms and simplifying file system design. Maintain a single unified address space. 

1) Are there any aspects that you think will make your OS design more complex than today's OSes? Please explain.

* The OS might need to incorporate wear leveling algorithms to distribute write and erase cycles evenly across the memory to prolong its lifespan.
* Persistent memory retains data even when powered off, increasing the risk of unauthorized data access. More encryption schemes might be needed to protect data.
* No need to use log (LRVM). 
* No need for page cache. 

### Question 2: MapReduce

Google's MapReduce has been arguably the most influential big data processing system to date.

1) Describe briefly how does MapReduce work. Is there any single point of failure in its design? Please explain.

* MapReduce is a programming model and an associated system for processing and generating large datasets. Map Function: The Map function takes an input pair and produces a set of intermediate key-value pairs. All intermediate values associated with the same intermediate key are grouped and passes them to the Reduce function. The reduce function merges these values together to form a possibly smaller set of values. 

* MapReduce has a single point of failure in the form of the **Master node**. The Master node is responsible for coordinating the Map and Reduce tasks, including task assignment, fault tolerance, and data distribution.

1) How does MapReduce handle worker failures? What are the implicit assumptions behind this technique? When does this technique work well and when it doesn't?

* Handle worker failure through task re-execution. The Master node periodically pings the worker nodes. Any map or reduce tasks that were in progress on the failed worker are rescheduled and executed on another available worker. Completed map tasks' output is stored on the local disk of the workers. The Master node keeps track of the locations of these outputs. If a worker holding the output fails, the tasks that generated those outputs are re-executed.

* Assumptions: (1) map and reduce tasks are independent. (2) There are enough workers available to take over the failed tasks. 

* Works well: tasks are small and can be re-executed quickly. It doesn't work well when there are heavy interdependencies between tasks, or the tasks are very large or long-running, the cost of re-executing them due to failures can be high.

1) What are the challenges of supporting iterative computations such as Machine Learning on top of MapReduce? Give a solution to address these challenges. 

* For iterative computations, it requires multiple passes of the same data. MapReduce is designed for single-pass computations, so data needs to be reloaded from the distributed file system for each iteration, leading to significant I/O overhead. 

* Spark. Spark keeps data in memory between iterations, which significantly reduces the I/O overhead compared to loading data from disk for each iteration.

### Question 3: Storage Systems

We are building a storage system in which the actual storage is somewhat distinct and possibly subject to tampering.

1) How can we make sure that files we access have not been modified?

* Cryptographic hashes of the file when it is first stored. Verify it is the same file by comparing the hashes. 
  * Compute a cryptographic hash (e.g., SHA-256) of each file when it is initially stored. 
  * Store these hashes securely, possibly in a tamper-proof database (once written, cannot be altered or deleted) or secure ledger.
  * Each time a file is accessed, recompute its hash and compare it with the stored hash. If the hashes match, the file has not been modified.

1) Now we would like to load the file lazily using demand paging. What problems does this introduce and how might we address them?

* Verifying the integrity of the entire file upon each page load can be inefficient. If only a portion of the file is loaded initially, tampering with unloaded parts may not be detected immediately.

1) If we are storing multiple copies anyway for disaster tolerance, how might this help us with tampering?

* Detect inconsistencies among multiple copies by directly comparing the content or the cryptographic hashes of the files.

4) Assuming a block-device model, what changes might we make to support these kinds of secure accesses?

* Use additional metadata blocks to store hashes and digital signatures for blocks. This might involve creating a hierarchical structure similar to **Merkle trees**.


### Alternative answers from Grok-2

# Secure Storage System Design Considerations

## 1) Ensuring File Integrity

**Solution: Cryptographic Hashing**

- **Method:** Use cryptographic hash functions (like SHA-256) to generate a unique hash for each file. Store these hashes securely, either in a separate, tamper-proof storage or in a blockchain for immutability.
  
- **Verification:** When accessing a file, compute its hash on-the-fly and compare it with the stored hash. If they match, the file has not been modified.

- **Additional Measures:**
  - **Digital Signatures:** Use digital signatures to verify the authenticity of files. This ensures not only integrity but also that the file comes from a trusted source.
  - **HMAC:** Use Hash-based Message Authentication Codes to verify both the integrity and authenticity of data.

## 2) Lazy Loading with Demand Paging

**Problems Introduced:**

- **Security:** Loading parts of a file on demand means that the entire file isn't immediately checked for integrity.
- **Consistency:** If parts of the file are modified or tampered with after initial loading, subsequent pages might be inconsistent.

**Addressing These Issues:**

- **Partial Hash Verification:** 
  - Compute and store hashes for each page or block of the file. When a page is loaded, verify its hash before use.

- **Delayed Integrity Check:**
  - Perform a full integrity check when the file is completely loaded or when certain conditions are met (e.g., after a certain number of pages have been accessed).

- **Secure Paging:**
  - Implement secure demand paging where each page load includes an integrity check. This might introduce some latency but ensures ongoing security.

## 3) Multiple Copies for Disaster Tolerance

**Leveraging for Tamper Detection:**

- **Redundancy for Verification:** 
  - Store multiple copies of the file in different locations. Compare the hashes of these copies periodically or upon access to detect tampering.

- **Consensus Mechanism:**
  - If multiple copies exist, use a majority vote or consensus mechanism to determine the correct version of the file if discrepancies are found.

- **Distributed Hash Tables (DHT):**
  - Use DHT to distribute file hashes across the network, making tampering more difficult as it would require altering multiple distributed hash entries.

## 4) Block-Device Model Changes for Secure Access

**Modifications:**

- **Encrypted Storage:**
  - Implement block-level encryption. Each block is encrypted, and only decrypted when needed, adding a layer of security against tampering.

- **Integrity Metadata:**
  - Add metadata to each block that includes a hash of the block's content. This metadata can be stored in a separate area or integrated into the block structure.

- **Journaling:**
  - Use journaling to log all write operations. This can help in reconstructing the original state if tampering is detected.

- **Immutable Block Design:**
  - Design blocks to be immutable once written. Any change requires creating a new block, which can be tracked through a blockchain-like structure for each file or block.

- **Access Control:**
  - Implement strict access controls at the block level, ensuring that only authorized operations can modify or read blocks.

- **Versioning:**
  - Maintain versions of blocks. If tampering is detected, you can roll back to a previous, verified version.

**Implementation Considerations:**

- **Performance vs. Security:** Balance the need for security with performance. Techniques like partial hash verification or delayed integrity checks can mitigate performance impacts.

- **Scalability:** Ensure that the security measures scale with the storage system, considering both the number of files and the size of each file.

- **Audit Trails:** Keep detailed logs of all access and modifications, which can be crucial for forensic analysis in case of security breaches.

By implementing these strategies, you can enhance the security of your storage system against tampering, while also managing the complexities introduced by demand paging and leveraging redundancy for both disaster recovery and security purposes.


## (Spring 2012 - Joseph & Culler):

### Time and Factors in Distributed Systems

A key factor that differentiates distributed systems from systems running on a single computer is the lack of global time.  Each of the components in a distributed system typically maintains its local clock and timestamps of various sorts are exchanged in messages. An obvious example is the Network Time Protocol (NTP). Unfortunately,
the communication of clocks uses essentially the same basic mechanisms, and suffers the same vagaries, as all other information transfer within the distributed system, and this places certain fundamental limits on distributed system operation.

* Briefly outline how clocks and/or timestamps are used at various
	levels of distributed systems, such as the transport layer, the
	filesystem, or applications (e.g., web applications).

* Transport layers: RTT. or sequence numbers and acknowledgements.
* File System: file timestamps (creation, modification, access time). Using versioning for consistency.
* Applications: Timestamps used to track user sections and timeouts. Used for logging and monitroing events. 
* There has been quite a bit of progress in reducing the end-to-end latency of communication networks and increasing the transfer bandwidth. What impact does this have on the uses that you have outlined?

* Improved synchronization for clocks (RTT).
* Enhanced real-time applications (e.g. video conferencing and gaming. )

* What additional benefits would derive from improving the predictability of communication behavior, say by improving the worst-case (as opposed to the best-case) latency?  Why is this kind of guarantee hard?

* Better reliability and more predictable performance. 
* Challenges: network variability (network load, congestion, hardware performance). Environmental factors (electromagnetic interference). Complex network protocols. 
*  What measures can be taking in the distributed algorithm, i.e., the protocol to overcome faults, foibles, and limitations in currently deployed systems and networks?
* Consensus Algorithms: Algorithms like Paxos and Raft ensure consensus among distributed nodes even in the presence of faults.
* Replication and Redundancy: Replicating data and services across multiple nodes helps maintain availability and consistency despite individual node failures.

5.	Overall, would you prefer improvements in maximum performance or predictability?

* Preditability. Real-time and mission-critical applications benefit more from predictability, as consistent performance is crucial.  Predictability can simplify system design, making it easier to guarantee service levels and plan resource allocation. Predictability often results in a smoother user experience, as users perceive consistent performance as more reliable.

Hint: Sometimes formalizing performance characteristics can simplify discussion of system design trade-offs.

### Disconnected Operation in File Systems

In this question, we'll explore the challenges of associated with intermittent connectivity in the Coda client-server distributed file system.

* Explain the advantages and design choices for Coda versus using a version control system, such as SVN or git

* Coda is specifically designed to handle intermittent connectivity. It allows users to continue working with files even when disconnected from the network. This is achieved through aggressive caching and disconnected operations, where clients can access and modify cached copies of files.
* SVN or Git is mostly used for version control. They are optimized for version control, collaboration, and tracking changes over time, 
* Real time synchronization: real-time access and sync, unlike version control system where explicit actions are needed to push or pull changes 
* Transparency: seamless file system experience, making distribution aspect transparent to the user
* Coda provides conflict resolution with server. While version control is Commit based and requires manual conflict resolutions.
  * Version vectors can be used to detect conflicts.

* Briefly outline how Coda uses caching to improve performance and scalability.

* Each client maintains a local cache of files and directories that have been accessed. This allows users to quickly read and write to files without needing constant communication with the server. When a client goes offline, it can continue to operate on the cached files. Any changes made while offline are logged locally and synchronized with the server once the connection is reestablished.
*  Coda uses **callback-based cache coherence**, and **whole-file caching**
* Suppose multiple disconnected Coda clients update the same file.
	Explain what Coda does when the clients reconnect with the server.
* Use optimistic replica control. Propagates changes back to servers (i.e. transfer updates, resolves conflicts). On conflicts, users use a debugging program to replay selectively
* W-W conflict is uncommon; low degree of write share in UNIX.

4.	Describe how you would change Coda to support a Dropbox or box.net
	model of cloud-based DFS

* Transition to a cloud storage backend, using services like Amazon S3
* Current Coda requires manual reintegration. Implement automatic synchronization of files between the local device and the cloud. Use efficient conflict resolution strategies to handle simultaneous modifications (e.g., versioning, conflict resolution dialogs).
* Implement real-time collaboration features similar to DropBox. 

### Scheduling, Interrupt Handling, and Scaling in Multicore Systems

In this problem we will explore system design trade-offs and the interplay of OS and architecture in the specific context of multicore systems.

* In a Multicore system, what measures can be taken in the operating system to maximize the effectiveness of caches? When do such measures work particularly well? And when do they introduce problems?  What measures can be taken to overcome these problems? Are there any specific solutions that you are familiar with?

* Processor affinity: Assigning threads to specific cores to maximize cache locality. This works well when threads have predictable access patterns and data dependencies.
* Cache-aware scheduling: Scheduling threads based on cache affinity to reduce cache misses. This can introduce problems when threads have conflicting cache access patterns or when the cache is shared between cores.
* Cache partitioning: Allocating cache space to specific cores or threads to reduce contention. This can help overcome cache sharing problems but may introduce complexity in cache management and resource allocation.
* Prefetching. 

* If the thread scheduler is protected by a lock, what is the maximum
	rate that threads can be scheduled onto cores?  What can be done to
	improve the scalability of the scheduler?  Are there any specific
	approaches that you can name?

* Time: 1 / (time to acquire lock + time to schedule thread + context switch)
* Per-Core Scheduling Queues. 
* Linux Completely Fair Scheduler (CFS)

* When an interrupt occurs, which core services the interrupt?
	What measures can be taken to improve the scalability of interrupt
	processing?  Of these, which require hardware support and which
	can be done entirely in software? 

* An available core?

* Interrupt Controller: The interrupt controller (e.g. APIC) determines which core handles the interrupt.


4.	Network I/O tends to be the most demanding of interrupt servicing?
	In particular, we want to be able to scale the TCP/IP bandwidth as we
	increase the processing capabilities.  Besides just a faster link
	rate (1 gig and 10 gig ethernet will stay for a while) what measures
	can be taken to scale up the transmit bandwidth?  What additional
	measures are required to scale up the receive bandwidth?

* ? 
* Compression. 	


**There are a lot of older questions omitted.**