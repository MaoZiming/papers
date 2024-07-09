# UCB Prelim Questions

## (Fall 2015 - Stoica & Brewer):

### Question 1: Non-volatile fast storage

Recently IBM and Micron have introduced a revolutionary storage technology, 
called 3D Xpoint, which promises non-volatility as well as a performance close 
to RAM's performance, i.e., just 2.5x higher access latency and 50% lower 
memory bandwidth than today's DRAM. Additionally, 3D XPoint can provide up 
to 10x higher density than DRAM.

Assume you are asked to develop a new OS for a machine that uses exclusively 
3D XPoint technology, i.e., no disk drives, and no SSDs. 

1) Discuss how you can simplify the design of your OS? Think about files 
system, virtual memory, etc.

> The OS can treat 3D XPoint as a single large pool of memory, eliminating the need for separate memory management for RAM and storage management for disks/SSDs. Files can be accessed directly from 3D XPoint, reducing the need for extensive caching mechanisms and simplifying file system design. Maintain a single unified address space. 

1) Are there any aspects that you think will make your OS design more 
complex than today's OSes? Please explain.

> The OS might need to incorporate wear leveling algorithms to distribute write and erase cycles evenly across the memory to prolong its lifespan.
> Persistent memory retains data even when powered off, increasing the risk of unauthorized data access. More encryption schemes might be needed to protect data.

### Question 2: MapReduce

Google's MapReduce has been arguably the most influential big data processing 
system to date.

1) Describe briefly how does MapReduce work. Is there any single point of 
failure in its design? Please explain.

> MapReduce is a programming model and an associated system for processing and generating large datasets. Map Function: The Map function takes an input pair and produces a set of intermediate key-value pairs. All intermediate values associated with the same intermediate key are grouped and passes them to the Reduce function. The reduce function merges these values together to form a possibly smaller set of values. 

> MapReduce has a single point of failure in the form of the Master node. The Master node is responsible for coordinating the Map and Reduce tasks, including task assignment, fault tolerance, and data distribution.

1) How does MapReduce handle worker failures? What are the implicit 
assumptions behind this technique? When does this technique work well 
and when it doesn't?

> Handle worker failure through task re-execution. The Master node periodically pings the worker nodes. Any map or reduce tasks that were in progress on the failed worker are rescheduled and executed on another available worker. Completed map tasks' output is stored on the local disk of the workers. The Master node keeps track of the locations of these outputs. If a worker holding the output fails, the tasks that generated those outputs are re-executed.

> Assumptions: (1) map and reduce tasks are independent. (2) There are enough workers available to take over the failed tasks. 

> Works well: tasks are small and can be re-executed quickly. It doesn't work well when there are heavy interdependencies between tasks, or the tasks are very large or long-running, the cost of re-executing them due to failures can be high.

1) What are the challenges of supporting iterative computations such as 
Machine Learning on top of MapReduce? Give a solution to address these 
challenges. 

> For iterative computations, it requires multiple passes of the same data. MapReduce is designed for single-pass computations, so data needs to be reloaded from the distributed file system for each iteration, leading to significant I/O overhead. 

> Spark. Spark keeps data in memory between iterations, which significantly reduces the I/O overhead compared to loading data from disk for each iteration.

### Question 3: Storage Systems

We are building a storage system in which the actual storage is somewhat
distinct and possibly subject to tampering.

1) How can we make sure that files we access have not been modified?

> Cryptographic hashes of the file when it is first stored. Verify it is the same file by comparing the hashes. 

2) Now we would like to load the file lazily using demand paging. What problems does this introduce and how might we address them?

> Verifying the integrity of the entire file upon each page load can be inefficient. If only a portion of the file is loaded initially, tampering with unloaded parts may not be detected immediately.

3) If we are storing multiple copies anyway for disaster tolerance, how might 
this help us with tampering?

> Detect inconsistencies among multiple copies by directly comparing the content or the cryptographic hashes of the files.

4) Assuming a block-device model, what changes might we make to support 
these kinds of secure accesses?

> Use additional metadata blocks to store hashes and digital signatures for blocks. This might involve creating a hierarchical structure similar to Merkle trees.

## (Spring 2012 - Joseph & Culler):

### Time and Factors in Distributed Systems

A key factor that differentiates distributed systems from systems
running on a single computer is the lack of global time.  Each of
the components in a distributed system typically maintains its local
clock and timestamps of various sorts are exchanged in messages. 
An obvious example is the Network Time Protocol (NTP). Unfortunately,
the communication of clocks uses essentially the same basic mechanisms,
and suffers the same vagaries, as all other information transfer within 
the distributed system, and this places certain fundamental limits on
distributed system operation.

1.	Briefly outline how clocks and/or timestamps are used at various
	levels of distributed systems, such as the transport layer, the
	filesystem, or applications (e.g., web applications).

> Transport layers: RTT. or sequence numbers and acknowledgements.

> File System: file timestamps (creation, modification, access time). Using versioning for consistency.

> Applications: Timestamps used to track user sections and timeouts. Used for logging and monitroing events. 

2.	There has been quite a bit of progress in reducing the end-to-end
	latency of communication networks and increasing the transfer
	bandwidth. What impact does this have on the uses that you have
	outlined?

> Improved synchronization for clocks (RTT).

> Enhanced real-time applications (e.g. video conferencing and gaming. )

3.	What additional benefits would derive from improving the
	predictability of communication behavior, say by improving the
	worst-case (as opposed to the best-case) latency?  Why is this
	kind of guarantee hard?

> Better reliability and more predictable performance. 

> Challenges: network variability (network load, congestion, hardware performance). Environmental factors (electromagnetic interference). Complex network protocols. 

4.  What measures can be taking in the distributed algorithm, i.e.,
	the protocol to overcome faults, foibles, and limitations in
	currently deployed systems and networks?

> Consensus Algorithms: Algorithms like Paxos and Raft ensure consensus among distributed nodes even in the presence of faults.

> Replication and Redundancy: Replicating data and services across multiple nodes helps maintain availability and consistency despite individual node failures.

5.	Overall, would you prefer improvements in maximum performance or
	predictability?

> Preditability. Real-time and mission-critical applications benefit more from predictability, as consistent performance is crucial.  Predictability can simplify system design, making it easier to guarantee service levels and plan resource allocation. Predictability often results in a smoother user experience, as users perceive consistent performance as more reliable.

Hint: Sometimes formalizing performance characteristics can simplify
discussion of system design trade-offs.

### Disconnected Operation in File Systems

In this question, we'll explore the challenges of associated with
intermittent connectivity in the Coda client-server distributed file system.

1.	Explain the advantages and design choices for Coda versus using
	a version control system, such as SVN or git

> Coda is specifically designed to handle intermittent connectivity. It allows users to continue working with files even when disconnected from the network. This is achieved through aggressive caching and disconnected operations, where clients can access and modify cached copies of files.

> SVN or Git is mostly used for version control. They are optimized for version control, collaboration, and tracking changes over time, 

2.	Briefly outline how Coda uses caching to improve performance
	and scalability.

> Each client maintains a local cache of files and directories that have been accessed. This allows users to quickly read and write to files without needing constant communication with the server. When a client goes offline, it can continue to operate on the cached files. Any changes made while offline are logged locally and synchronized with the server once the connection is reestablished.

>  Coda uses **callback-based cache coherence**, and **whole-file caching**

3.	Suppose multiple disconnected Coda clients update the same file.
	Explain what Coda does when the clients reconnect with the server.

> Use optimistic replica control. Propagates changes back to servers (i.e. transfer updates, resolves conflicts). On conflicts, users use a debugging program to replay selectively

> W-W conflict is uncommon; low degree of write share in UNIX.

4.	Describe how you would change Coda to support a Dropbox or box.net
	model of cloud-based DFS

> Transition to a cloud storage backend, using services like Amazon S3

> Current Coda requires manual reintegration. Implement automatic synchronization of files between the local device and the cloud. Use efficient conflict resolution strategies to handle simultaneous modifications (e.g., versioning, conflict resolution dialogs).

### Scheduling, Interrupt Handling, and Scaling in Multicore Systems

In this problem we will explore system design trade-offs and the
interplay of OS and architecture in the specific context of multicore systems.

1.	In a Multicore system, what measures can be taken in the operating
	system to maximize the effectiveness of caches? When do such measures work particularly well? And when do they introduce problems?  What measures can be taken to overcome these problems? Are there any specific solutions that you are familiar with?

> Processor affinity: Assigning threads to specific cores to maximize cache locality. This works well when threads have predictable access patterns and data dependencies.
> Cache-aware scheduling: Scheduling threads based on cache affinity to reduce cache misses. This can introduce problems when threads have conflicting cache access patterns or when the cache is shared between cores.
> Cache partitioning: Allocating cache space to specific cores or threads to reduce contention. This can help overcome cache sharing problems but may introduce complexity in cache management and resource allocation.
> Prefetching. 

2.	If the thread scheduler is protected by a lock, what is the maximum
	rate that threads can be scheduled onto cores?  What can be done to
	improve the scalability of the scheduler?  Are there any specific
	approaches that you can name?

> Time: 1 / (time to acquire lock + time to schedule thread + context switch)
> Per-Core Scheduling Queues. 
> Linux Completely Fair Scheduler (CFS)

3.	When an interrupt occurs, which core services the interrupt?
	What measures can be taken to improve the scalability of interrupt
	processing?  Of these, which require hardware support and which
	can be done entirely in software? 

> An available core?

> Interrupt Controller: The interrupt controller (e.g. APIC) determines which core handles the interrupt.


4.	Network I/O tends to be the most demanding of interrupt servicing?
	In particular, we want to be able to scale the TCP/IP bandwidth as we
	increase the processing capabilities.  Besides just a faster link
	rate (1 gig and 10 gig ethernet will stay for a while) what measures
	can be taken to scale up the transmit bandwidth?  What additional
	measures are required to scale up the receive bandwidth?

> ? 

## (Spring 2005 - Brewer & Culler):
"The exam contained three questions centered on topics of error handling,
driver isolation and migration.  However, each question provided a chance
to demonstrate broad knowledge of aspects of operating systems."

## (Fall 2004 - Smith & Wagner):
"The exam consisted of 3 questions.  The first concerned the interface 
between the operating system and the computer architecture. The second 
involved distributed system design, tradeoffs and optimizations.  The 
third concerned virtual machine design and operation."

## (Fall 2003 - Brewer & Joseph):
"The exam contained three questions centered on 
(1) disk read/write caching and filesystem interactions,
(2) segments and paging issues, and 
(3) issues in migrating a centralized server to a decentralized, fault-tolerant environment.  

In each case the question started from basic knowledge and extended toward open-ended design."

## (Fall 2002 - Katz & Joseph):
"The exam consisted of the following topics: 
1) How do you build a transaction for disk writes? (only single page 
   writes are atomic)
2) How do you design a system for airplane collision prevention? (distributed 
   systems problem with some reliability challenges)
3) How would you build a new streaming protocol? (application- and OS-level networking)  

## (Fall 2001 - Smith & Wagner):
"The exam consisted of questions regarding: memory management, fault
tolerance, viruses, and network-attached storage."

(Fall 2000 - Smith & Brewer):
"The exam consisted of three questions regarding: (1) paging,
(2) virtual machines, and (3) RPC."

## (Spring 2000 - Katz & Joseph):
"The exam consisted of three questions centered on (1) reliability,
availability, and consistency, (2) time management, ordering,
synchronization, and communication in distributed systems, and (3)
redirection in operating systems and networking.  In each case, the
question started from basic knowledge and extended towards open-ended
design."

## (Fall 1999 - Culler & Brewer):
"Q1: Microkernel architectures were traditionally designed with
distributed computation and high-performance servers in mind.  These
days, however, microkernels are almost never used in these situations.
Microkernels are making a comeback in the development of small devices.
Describe the properties of microkernels that might contribute to these
phenomenon.

Q2: Logs, what are they good for?  And transaction?  (Fishing for: ACID
applications, batched writes, auditing, etc.)

Q3: Suppose you were interested in accessing network services (e.g.,
Amazon.com, information about a room you just entered, etc.) while
protecting your anonymity.  How would you do this?  How would you
support persistent sessions?  How would you prevent different services
from correlating your actions to deduce additional information about
you?"

## (Spring 1999 - Brewer & Joseph):
"The exam consisted of three questions centered on (1) the impact of
technology trends on the design of distributed computing systems, (2)
the benefits and drawbacks to microkernels, and (3) the construction of
an RPC protocol that ensures exactly once execution.  In each case, the
question started from basic knowledge and extended towards open-ended
design."

## (Fall 1998 - Culler & Joseph):
"The exam consisted of three questions centered on (1) the impact of
technology trends on operating system design, (2) time management,
ordering, synchronization, and communication in distributed systems, and
(3) security in financial systems with smart user devices.  In each
case, the question started from basic knowledge and extended towards
open-ended design."
