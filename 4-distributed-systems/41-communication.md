# Distributed Systems

## DSM

* Distributed shared memory (DSM) systems enable processes on different machines to share a large, virtual address space
  * Through virtual memory system of the OS 
  * Not widely used because
    * Failure handling is hard
      * machine fails, data structure unavailable, addr space missing, etc. 
    * Performance: remote fetching 

## RPC

### Stub Generator
  * The input to such a compiler is simply the set of calls a server wishes to export to clients. Conceptually, it could be something as simple as this:

```c
interface {
  int func1(int arg1);
  int func2(int arg1, int arg2);
};
```
  * For the client, a client stub is generated, which contains each of the functions specified in the interface; a client program wishing to use this RPC service would link with client stub and call into it in order to make RPCs.
  * To the client, the code just appears as a function call (e.g., the client calls `func1(x)`); internally, the code in the client stub for `func1()` does:
    * Create a message buffer. A message buffer is usually just a contiguous array of bytes of some size.
    * Pack the needed information into the message buffer: marshaling of arguments or the serialization of the message.
    * Send the message to the destination RPC server.
    * Wait for the reply. Because function calls are usually synchronous, the call will wait for its completion.
    * Unpack return code and other arguments. This step is also known as unmarshaling or deserialization.
    * Return to the caller. Finally, just return from the client stub back into the client code.

### Run-time library

* The run-time must also handle procedure calls with large arguments, larger than what can fit into a single packet. Some lower-level network protocols provide such sender-side fragmentation (of larger packets into a set of smaller ones) and receiver-side reassembly (of smaller parts into one larger logical whole).
* One issue: byte ordering. 
  * Big endian ordering vs. little endian ordering. 
  * Big endian stores bytes (say, of an integer) from most significant to least significant bits, much like Arabic numerals; little endian does the opposite. 
* Server uses thread pool to manage concurrency. 
    * Threads are created when server starts
    * When message arrives, dispatched to one of the worker threads 
    * Main thread keeps receiving data 


### Reliable Transport: UDP v.s TCP

* **UDP (User Datagram Protocol) - Unreliable Communication Layer** 
    * API
        * Reads and writes over socket API (FDs) 
        * Messages sent from / to ports to target a process on machine 
    * Provide minimal reliability features 
        * Messages may be lost, reordered, duplicated. 
        * Only protection: checksums to ensure data not corrupted (i.e. for integrity) 
* **TCP (Transmission Control Protocol) - Reliable Communication Layer** 
    * Use software to build reliable logical connections over unreliable physical connections 
        * No duplicates, message received exactly once 
    * Scheme
        * ACKS: Receiver send ack upon receiving messages 
        * TIMEOUT: Sender timeout when not receiving ack, and retry 
            * Adaptive timeouts 
        * SEQUENCE COUNTER 
            * Senders give each message an increasing unique sequence number
            * Receiver knows it has seen all messages before $N$ 
            * Suppose message $K$  is received 
                * If $K <= N,$ Msg $K$  is already delivered, ignore it 
                * If $K = N +1$, first time seeing this message 
                * If $K > N+1$, buffer this message so arrive in order 



## System Model

### Network 
* Reliable: Assumes all messages will be successfully delivered.
* Fair-loss: Assumes messages may be lost but will eventually get through.
* Arbitrary: No assumptions about message delivery.
### Nodes 
* Crash: Nodes will stop and not recover.
   * Similar to fail-stop. However, in a fail-stop scenario, the node intentionally halts its execution due to a detected error or by following a protocol for shutting down.
* Crash-recovery: Nodes can recover after a crash.
* Byzantine: Nodes can act arbitrarily, including maliciously.
* **Timing:** synchronous, partially synchronous, or asynchronous
    * Synchronous: message latency no greater than a known upper bound
    * Partial synchronous: async for some finite (but unknown) period of time, sync otherwise
      * Since the algorithm relies on predictable communication to detect and replace faulty primaries, intermittent synchrony (as in the case with partial synchrony) can lead to prolonged periods of inactivity or stalled consensus.
    * > In one version of partial synchrony, fixed bounds Δ and Φ exist, but they are not known a priori. The problem is to design protocols that work correctly in the partially synchronous system regardless of the actual values of the bounds Δ and Φ. In another version of partial synchrony, the bounds are known, but are only guaranteed to hold starting at some unknown time T, and protocols must be designed to work correctly regardless of when time T occurs.
    * Eventual synchrony: Starts asynchronous but eventually becomes and **remains** synchronous.
    * Async: messages can be delayed arbitrarily, nodes can pause execution arbitrarily, no timing guarantees at all
  * Paxos and Raft assumes partially synchronous networks. PBFT assumes eventual synchrony for liveness, and asynchrony for safety. 

* **Happens before relation**
    * An event is something happening at one node (sending, or receiving message, or local execution step)
    * We says event $a$ happens before event $b$ (written $a \rightarrow b$) iff
        * $a$ and $b$ occurred at the same node, and $a$ occurred before $b$ in that node’s local execution order
        * Or event $a$ is sending some message $m$, and event $b$ is receiving that same mesage $m$ (assuming sent messages are unique)
        * There exists some event $c$ such that $a \rightarrow c$ and $c \rightarrow b$
        * Partial order: it is possible that $a \rightarrow b$ nor $b \rightarrow a$, in that case $a$ and $b$ are concurrent (i.e. $a || b)$
* **Causality**
    * Taking from physics (relativity)
        * When $a \rightarrow b$, then $a$ **might have caused** $b$
        * When $a || b$, we know that $a$ **cannot have caused** $b$
    * Happens before relation encodes **potential causality**

* ![alt text](images/41-communication/broadcast-hierarchy.png)
  * FIFO broadcast extends reliable boadcast with the FIFO ordering guarantee. 
  * Causal broadcast ensures that messages are delivered respecting their causal dependencies, based on the happened-before relationship defined by Lamport.
    * Since the happened-before relationship includes the order in which messages are sent by a single process, causal broadcast naturally enforces FIFO ordering for messages from the same sender.
  * Total order broadcast (also known as atomic broadcast) ensures that all messages are delivered to all processes in the same total order.
    * **This allows for multiple senders**
  * FIFO Total Order Broadcast: Combines FIFO and total order guarantees, ensuring that messages from the same sender are delivered **in order** and all messages are delivered **in the same total order** across all processes.

* Choice of retry semantics
    * **At-most-once**: send request, don’t retry, update may not happen
    * **At-least-once**: retry request until ed ACK, may repeat update
    * **Exactly-once**: retry + idempotence or deduplication

## State machine replication

* Broadcast protocol to do replication!
* Total order broadcast: every node delivers the **same messages** in **same order**
* **State machine replication (SMR)**
    * **FIFO-total order** broadcast
    * Replica delivers update message: apply it to own state
    * Applying an update is deterministic
        * Each replica is state machine, state is all data it’s stored
            * Go through same sequence of state transitions in the same order
            * Ended up in the same state

### Consensus system models

* Paxos, Raft, etc. assume a **partially synchronous, crash-recovery** system model
  * It can be designed for crash-stop, but needs additional care for state persistence and quorum adjustments. 
* Why not async?
    * **FLP result**: there is no deterministic consensus algorithm that is guaranteed to terminate in an async crash-stop system model
    * Paxos, Raft, etc. use clocks only used for timeouts / failure detector to ensure progress. Safety (correctness) does not depend on timing.

### Leader election

* Multi-paxos, Raft, etc. use a leader to sequence messages
    * Use a **failure detector** (timeout) to determine suspected crash or unavailability of leader
    * On suspected leader crash, **elect a new one**
    * Prevent **two leaders at the same time** (”split-brain”)!

## SSL / TLS: a protocol, not a software package

* Standard method of communicating between processes in modern system is socket
* Secure Socket Layer (SSL)
    * Formally is insecure
    * Move encrypted data through an ordinary socket
    * I.e. set up a socket, set up a special structure to perform crypto, then hook the output of that structure to the input of the socket, reverse the process on the other end
    * Steps
        * Start negotiation between client and server
        * End in both sides finding some acceptable **set of ciphers and techniques** that balance between performance and security
            * I.e. Diffie-Hellman key exchange to create the key
* Transport Layer Security (TLS)
    * Use this
    * Evolved from SSL.
    * It should be noted that TLS does not secure data on end systems. It simply ensures the secure delivery of data over the Internet, avoiding possible eavesdropping and/or alteration of the content.
    * TLS is normally implemented on top of TCP in order to encrypt Application Layer protocols such as HTTP, FTP, SMTP and IMAP, although it can also be implemented on UDP, DCCP and SCTP as well 
    * However, SSL is an older technology that contains some security flaws. Transport Layer Security (TLS) is the upgraded version of SSL that fixes existing SSL vulnerabilities.

### Two Phased commits

![alt text](images/41-communication/2pc.png)

### Layering

![alt text](images/41-communication/five-layers.png)

– Physical: send bits
– Datalink: Connect two hosts on same physical media
– Network: Connect two hosts in a wide area network
– Transport: Connect two processes on (remote) hosts
– Applications: Enable applications running on remote hosts to interact

* TCP Flow control
  * – Avoid the sender over-flowing the receiver buffer
  * – Receiver only reads in-sequence data, and acks with the next sequence number is waiting for
  * – Sender never sends more data than the receiver can hold in its buffer 

### Consistency

Link: https://www.cs.princeton.edu/courses/archive/fall18/cos418/docs/p8-consistency.pdf


#### Linearizability / Atomic Consistency / Strict Consisntecy
* In essence, linearizability is a more stringent requirement that includes the guarantees of sequential consistency plus the real-time ordering of operations.
* Strict Consistency requires events to be ordered in the same real-time sequence in which they occurred. In other words, “an earlier write always has to be seen before a later write.” This is almost impossible to implement in distributed systems. Hence it remains in the realm of theoretical discussions.

```
Real-Time:   ---A1---A2---B1---B2---
Lineariz:    ---A1---A2---B1---B2--* (valid)
             ---A1---B1---A2---B2--* (valid)
             ---B1---A1---B2---A2--* (invalid, A1 < A2 in real-time)
```
* Importantly, linearizability acknowledges that there’s some time gap between when an operation is submitted to the system and when the system acknowledges it.
  * Only non-overlapping writes need to be ordered in a real-time sequence.
  * Not for writes/reads that overlap with each other. 
* **Some text suggests that strict consistency = linearizability.** Likely true, also as mentioned by ChatGPT. 
* ![alt text](images/41-communication/linearizability-serializability.png)

* Strict serializability: 
* * **Preserves real-time ordering**: Any transaction A that completes before transaction B begins, occurs before B in the total order.
* Strict serializability is a combination of serializability and linearizability
* Strict serializability ensures that the result of concurrently executing transactions is not only equivalent to some serial execution but also respects the real-time order of the transactions. This means that the system's behavior adheres to both the logical consistency of serializability and the real-time ordering guarantees of linearizability.

**IMPORTANT**
* Strict serializability: any transaction A that occurs after B.
* Linearizability: any operation A that occurs after B.
* In Linearizability, clients only have consistency guarantees for operations, where strict serializability allows clients to use transactions.

#### Serializability vs. Linearizability

**IMPORTANT**: serializability is NOT **strict serializability**. 
  * The first one is for ordering transactions in logical order. 
* **Serializability** is a global property; a property of an entire history of operations/transactions. 
  * Serializability mainly through 2PL. 
  * Conflict serializability:  requires that transactions do not have conflicting accesses to the same data item
  * View serializability only requires that transactions produce the same final result as a serial schedule.
* **Linearizability** is a local property; a property of a single operation/transaction. Another distinction is that linearizability includes a notion of real-time, which serializability does not

In Plain English
* Under linearizability, writes should appear to be instantaneous. Imprecisely, once a write completes, all later reads (where “later” is defined by wall-clock start time) should return the value of that write or the value of a later write. Once a read returns a particular value, all later reads should return that value or the value of a later write.
* Serializability is a guarantee about transactions, or groups of one or more operations over one or more objects. It guarantees that the execution of a set of transactions (usually containing read and write operations) over multiple items is equivalent to some serial execution (total ordering) of the transactions.

* ![alt text](images/41-communication/serializability-vs-linearizability.png)

#### Sequential Consistency
* **Order Maintenance**: The order of operations within each individual process is preserved.
* **Global Sequence**: There is a single global sequence of operations that all processes agree on, but it does not necessarily reflect the real-time order of operations.
* Sequential consistency: Preserves process ordering: All of a process’ operations appear in that order in the total order.
* Difference from linearizability? Sequence of ops across processes not determined by real-time


#### Causal Consistency
* Causal Consistency: Causal Consistency requires only related operations to have a global ordering between them. Two operations can be related because they acted on the same data item, or because they originated from the same thread.
* **Causal+ Consistency**: +: Replicas eventually converge

#### Consistency Prefix
* By requesting a consistent prefix, a reader is guaranteed to observe an ordered sequence of writes starting with
the first write to a data object.
* In other words, the reader sees a version of the data store that existed at the master at some time in the past. This is similar to the “snapshot isolation” consistency offered by many database management systems.

#### Bounded staleness
* Time-bounded staleness
* The storage system guarantees that a read operation will return any values written more than T minutes ago or more recently written values

#### Monotonic Reads
* With monotonic reads, a client can read arbitrarily stale data, as with eventual consistency, but is guaranteed to observe a data store that is increasingly up-to-date over time. In particular, if the client issues a read operation and then later issues another read to the same object(s), the second read will return the same value(s) or the results of later writes.

#### Read-my-writes
* A sequence of operations performed by a single client. 
* It guarantees that the effects of all writes that were performed by the client are visible to the client’s subsequent
reads.

* ![alt text](images/41-communication/baseball-consistency.png)



### Lamport Clock

* Each node maintains a counter that increments with each event. When nodes communicate, they update their counters based on the maximum value seen, ensuring a consistent order of events.
* Algorithm of Lamport Clocks:
  * Initialization: Each node initializes its clock LLL to 0.
  * Internal Event: When a node performs an internal event, it increments its clock LLL.
  * Send Message: When a node sends a message, it increments its clock LLL and includes this value in the message.
  * Receive Message: When a node receives a message with timestamp T: It sets L=max⁡(L,T)+1
* Advantages of Lamport Clocks:
  * Simple to implement and understand.
  * Ensures total ordering of events.

### Vector Clock

* Vector Clock is an algorithm that generates partial ordering of events and detects causality violations in a distributed system.
* Capture causal relationship
* This algorithm helps us label every process with a vector (a list of integers) with an integer for each local clock of every process within the system. So for N given processes, there will be vector/ array of size N. 
* How does the vector clock algorithm work: 
  * Initially, all the clocks are set to zero.
  * Every time, an Internal event occurs in a process, the value of the processes’s logical clock in the vector is incremented by 1
  * Also, every time a process sends a message, the value of the processes’s logical clock in the vector is incremented by 1.
  * Every time, a process receives a message, the value of the processes’s logical clock in the vector is incremented by 1, and moreover, each element is updated by taking the maximum of the value in its own vector clock and the value in the vector in the received message (for every element). 
* ![alt text](images/41-communication/vector-clock.png)
* To sum up, Vector clocks algorithms are used in distributed systems to provide a causally consistent ordering of events but the entire Vector is sent to each process for every message sent, in order to keep the vector clocks in sync.
* Advantages of Vector Clocks:
  * Accurately captures causality and concurrency.
  * Detects concurrent events, which Lamport clocks cannot do.

* Vector clock: network failures and message lost. 

### Matrix Clock
* Matrix clocks extend vector clocks by maintaining a matrix where each entry captures the history of vector clocks. This allows for more detailed tracking of causality relationships.

#### Components

* **Matrix Representation**: A matrix clock for a system with $n$ processes is represented as an $n \times n$ matrix. Let's denote this matrix for process $P_i$ as $MC_i$, where $MC_i[j, k]$ indicates the latest known clock value of process $P_k$ according to process $P_j$.

* **Row Representation**: Each row in the matrix corresponds to a vector clock for a process, and each column corresponds to the clock value known by a process for other processes.

## Rules for Maintenance

* **Local Event (Internal Event)**:
  * When process $P_i$ experiences an internal event, it increments its own clock value: $MC_i[i, i] += 1$.

* **Message Sending**:
  * When process $P_i$ sends a message to $P_j$, it includes its current matrix clock $MC_i$ in the message.

* **Message Receiving**:
  * When process $P_j$ receives a message from $P_i$ with the matrix clock $MC_i$, it updates its own matrix clock $MC_j$ as follows:
    * For each process $P_k$, update $MC_j[j, k]$ to be the maximum of $MC_j[j, k]$ and $MC_i[i, k]$.
    * Additionally, for the sender $P_i$, update $MC_j[i, i]$ to be the maximum of $MC_j[i, i]$ and $MC_i[i, i] + 1$.

### Two phased commit 

* Two-Phase Commit (2PC) is a distributed algorithm that ensures all participants in a distributed system agree on a transaction before it is committed
* Phases of Two-Phase Commit:
    * **Prepare Phase** (Voting Phase):
    * The coordinator sends a "**prepare**" request to all participants.
    * Each participant performs the necessary operations to ensure it can commit the transaction (e.g., locking resources, writing data to a log).
    * Participants respond with either "**vote-commit**" (if they can commit) or "**vote-abort**" (if they cannot commit).
* Commit Phase:
    * If all participants respond with "vote-commit," the coordinator sends a "commit" message to all participants, instructing them to commit the transaction.
    * If any participant responds with "vote-abort," the coordinator sends an "abort" message to all participants, instructing them to roll back the transaction.
    * Participants then perform the commit or abort operation and acknowledge back to the coordinator.
* How does the Two-Phase Commit protocol handle failures during the commit phase?
    * If a failure occurs during the commit phase, the coordinator must ensure that the transaction is completed once the system recovers. The protocol assumes that the coordinator and participants can eventually recover and re-attempt the commit or abort based on the decision recorded before the failure. This might involve logging or other recovery mechanisms.
* What is the primary disadvantage of the Two-Phase Commit protocol?
    * The primary disadvantage of the Two-Phase Commit protocol is its **blocking nature**. If the coordinator crashes after sending the prepare message but before sending the commit/abort decision, participants can be left in an uncertain state, blocking further transactions until the coordinator recovers. This can lead to inefficiencies and delays in distributed systems.
* Can be used for replication with strong consistency. 

## Weak Consistency

### Consistency Model of Dynamo

Amazon's Dynamo is a highly available and scalable key-value store, and its consistency model is based on the principles of **eventual consistency**. However, it provides tunable consistency, allowing applications to choose different consistency levels based on their requirements. Here’s a breakdown of Dynamo's consistency model:

1. **Eventual Consistency**: Dynamo ensures that if no new updates are made to a data item, eventually all replicas will converge to the same value. This means that reads may not return the most recent write immediately, but over time, the system guarantees consistency.

2. **Tunable Consistency**: Dynamo allows developers to adjust the consistency vs. availability trade-off by configuring two parameters:
   - **R (Read Quorum)**: The number of replicas that must agree on a value for a read to be considered successful.
   - **W (Write Quorum)**: The number of replicas that must acknowledge a write for it to be considered successful.

   By adjusting the values of `R`, `W`, and `N` (the total number of replicas), the system can be tuned for different levels of consistency:
   - **Strong Consistency**: Can be achieved by setting `R + W > N`, meaning that there is at least one replica overlap between read and write operations, ensuring that a read always sees the most recent write.
   - **Weaker Consistency**: By setting `R + W <= N`, Dynamo allows for higher availability and lower latency, at the cost of potentially stale reads.

3. **Vector Clocks and Conflict Resolution**: Dynamo uses vector clocks to track the causality of updates across replicas. When divergent versions of an object are detected, they are presented to the application for reconciliation. This means that conflicts are resolved at the application level, often by using techniques like last-write-wins (LWW) or custom reconciliation logic.

In summary, Dynamo follows an **eventual consistency** model with **tunable consistency**, allowing applications to choose between stronger consistency and higher availability.


# Strong Consistency Protocols in Distributed Systems

## 1. Paxos
- **Overview**: Paxos is a consensus protocol that ensures strong consistency by allowing a group of nodes to agree on a single value even in the presence of failures. Once a decision is made, it cannot be changed.
- **Usage**: Widely used in distributed databases and coordination services (e.g., Google Chubby, Google Spanner).
- **Trade-offs**: Complex to implement and introduces performance overhead due to multiple rounds of communication.

## 2. Raft
- **Overview**: Raft is a consensus algorithm similar to Paxos but designed to be more understandable and easier to implement. Raft elects a leader, and the leader replicates updates to other nodes.
- **Usage**: Used in systems like etcd, Consul, and Kubernetes for distributed consensus and leader election.
- **Trade-offs**: While simpler than Paxos, Raft still introduces latency during leader elections and failures.

## 3. Viewstamped Replication (VR)
- **Overview**: VR is a protocol for replicating services across nodes, ensuring strong consistency by using a primary node that manages updates. A new primary is elected if the original primary fails.
- **Usage**: Commonly used in replicated state machines and distributed databases.
- **Trade-offs**: Coordination overhead and complexity during view changes.

## 4. Zookeeper's Zab Protocol
- **Overview**: Zab (Zookeeper Atomic Broadcast) is a consensus protocol used by Zookeeper to ensure strong consistency. The leader manages updates, and followers replicate them in the same order.
- **Usage**: Specifically designed for Zookeeper, which provides distributed coordination services.
- **Trade-offs**: May experience delays during leader elections, like other leader-based protocols.

## 5. Multi-Paxos
- **Overview**: An extension of Paxos optimized for handling multiple decisions, with leader election to streamline the process of committing multiple commands.
- **Usage**: Used in distributed databases requiring a sequence of consistent decisions (e.g., Google Spanner, Microsoft Azure Cosmos DB).
- **Trade-offs**: More efficient than standard Paxos for multiple operations, but still carries communication overhead.

## 6. Chubby's Lease-Based Protocol
- **Overview**: Chubby, Google's distributed lock service, uses a lease-based protocol where a primary node manages updates. The primary holds a lease and renews it to maintain authority.
- **Usage**: Internally used at Google for distributed coordination, leader election, and lock management.
- **Trade-offs**: Lease-based systems rely on timely lease renewals, and network partitions or delays can complicate recovery.

## 7. Two-Phase Commit (2PC)
- **Overview**: 2PC is a distributed transaction protocol that ensures all participating replicas either commit or roll back a transaction in a coordinated manner, guaranteeing strong consistency.
- **Usage**: Commonly used in distributed databases, transaction processing systems, and services where atomic transactions are required.
- **Trade-offs**: 2PC suffers from the blocking problem, where replicas can be left in an uncertain state if the coordinator fails, impacting availability and performance.

## 8. Spanner’s TrueTime and Synchronous Replication
- **Overview**: Google Spanner combines the Paxos consensus algorithm with TrueTime, which provides globally synchronized timestamps to achieve strong consistency, even in globally distributed systems.
- **Usage**: Google Spanner is a globally distributed, strongly consistent database that supports SQL semantics and distributed transactions.
- **Trade-offs**: Spanner requires specialized hardware (atomic clocks and GPS) for TrueTime and has higher latencies due to synchronous replication.

## 9. Chain Replication
- **Overview**: Chain replication organizes nodes in a chain, where updates are sequentially propagated from the head to the tail. The tail node acknowledges commits, ensuring that all preceding nodes have applied the update.
- **Usage**: Used in systems requiring strong consistency with simpler replication models, such as distributed key-value stores.
- **Trade-offs**: Efficient for write-heavy workloads, but introduces latency for read operations if reads must be served by the tail for consistency.

## 10. Quorum-Based Protocols (e.g., Quorum Reads/Writes)
- **Overview**: Quorum-based protocols ensure strong consistency by requiring a majority of replicas to agree on a value before a read or write is considered successful. Overlapping reads and writes ensure consistency across distributed nodes.
- **Usage**: Used in distributed databases like Apache Cassandra (when configured for strong consistency) and DynamoDB (with appropriate R/W settings).
- **Trade-offs**: Quorum-based systems can experience increased latency due to the need to contact multiple replicas, and may reduce availability during network partitions or failures.

## 11. Primary-Backup Replication (with Strong Consistency Guarantees)
- **Overview**: In primary-backup replication, a primary node handles all updates, and backups replicate the primary's state. Strong consistency is ensured by routing all reads and writes through the primary, with synchronous updates to the backups.
- **Usage**: Used in distributed systems like databases, file systems, and services requiring strong consistency.
- **Trade-offs**: The primary can become a bottleneck, and failure of the primary requires a failover, which can introduce downtime or latency. Synchronous replication adds communication overhead.
