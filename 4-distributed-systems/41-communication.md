# Distributed Systems

## DSM

- Distributed shared memory (DSM) systems enable processes on different machines to share a large, virtual address space
  - Through virtual memory system of the OS 
  - Not widely used because
    - Failure handling is hard
      - machine fails, data structure unavailable, addr space missing, etc. 
    - Performance: remote fetching 

## RPC

### Stub Generator
  - The input to such a compiler is simply the set of calls a server wishes to export to clients. Conceptually, it could be something as simple as this:

```c
interface {
  int func1(int arg1);
  int func2(int arg1, int arg2);
};
```
  - For the client, a client stub is generated, which contains each of the functions specified in the interface; a client program wishing to use this RPC service would link with client stub and call into it in order to make RPCs.
  - To the client, the code just appears as a function call (e.g., the client calls `func1(x)`); internally, the code in the client stub for `func1()` does:
    - Create a message buffer. A message buffer is usually just a contiguous array of bytes of some size.
    - Pack the needed information into the message buffer: marshaling of arguments or the serialization of the message.
    - Send the message to the destination RPC server.
    - Wait for the reply. Because function calls are usually synchronous, the call will wait for its completion.
    - Unpack return code and other arguments. This step is also known as unmarshaling or deserialization.
    - Return to the caller. Finally, just return from the client stub back into the client code.

### Run-time library

- The run-time must also handle procedure calls with large arguments, larger than what can fit into a single packet. Some lower-level network protocols provide such sender-side fragmentation (of larger packets into a set of smaller ones) and receiver-side reassembly (of smaller parts into one larger logical whole).
- One issue: byte ordering. 
  - Big endian ordering vs. little endian ordering. 
  - Big endian stores bytes (say, of an integer) from most significant to least significant bits, much like Arabic numerals; little endian does the opposite. 
- Server uses thread pool to manage concurrency. 
    1. Threads are created when server starts
    2. When message arrives, dispatched to one of the worker threads 
    3. Main thread keeps receiving data 


### Reliable Transport: UDP v.s TCP

1. **UDP (User Datagram Protocol) - Unreliable Communication Layer** 
    1. API
        1. Reads and writes over socket API (FDs) 
        2. Messages sent from / to ports to target a process on machine 
    2. Provide minimal reliability features 
        1. Messages may be lost, reordered, duplicated. 
        2. Only protection: checksums to ensure data not corrupted (i.e. for integrity) 
2. **TCP (Transmission Control Protocol) - Reliable Communication Layer** 
    1. Use software to build reliable logical connections over unreliable physical connections 
        1. No duplicates, message received exactly once 
    2. Scheme
        1. ACKS: Receiver send ack upon receiving messages 
        2. TIMEOUT: Sender timeout when not receiving ack, and retry 
            1. Adaptive timeouts 
        3. SEQUENCE COUNTER 
            1. Senders give each message an increasing unique sequence number
            2. Receiver knows it has seen all messages before $N$ 
            3. Suppose message $K$  is received 
                1. If $K <= N,$ Msg $K$  is already delivered, ignore it 
                2. If $K = N +1$, first time seeing this message 
                3. If $K > N+1$, buffer this message so arrive in order 



## System Model

### Network 
1. Reliable: Assumes all messages will be successfully delivered.
2. Fair-loss: Assumes messages may be lost but will eventually get through.
3. Arbitrary: No assumptions about message delivery.
### Nodes 
1. Crash: Nodes will stop and not recover.
   1. Similar to fail-stop. However, in a fail-stop scenario, the node intentionally halts its execution due to a detected error or by following a protocol for shutting down.
2. Crash-recovery: Nodes can recover after a crash.
3. Byzantine: Nodes can act arbitrarily, including maliciously.
- **Timing:** synchronous, partially synchronous, or asynchronous
    - Synchronous: message latency no greater than a known upper bound
    - Partial synchronous: async for some finite (but unknown) period of time, sync otherwise
      - Since the algorithm relies on predictable communication to detect and replace faulty primaries, intermittent synchrony (as in the case with partial synchrony) can lead to prolonged periods of inactivity or stalled consensus.
    - > In one version of partial synchrony, fixed bounds Δ and Φ exist, but they are not known a priori. The problem is to design protocols that work correctly in the partially synchronous system regardless of the actual values of the bounds Δ and Φ. In another version of partial synchrony, the bounds are known, but are only guaranteed to hold starting at some unknown time T, and protocols must be designed to work correctly regardless of when time T occurs.
    - Eventual synchrony: Starts asynchronous but eventually becomes and **remains** synchronous.
    - Async: messages can be delayed arbitrarily, nodes can pause execution arbitrarily, no timing guarantees at all
  - Paxos and Raft assumes partially synchronous networks. PBFT assumes eventual synchrony for liveness, and asynchrony for safety. 

- **Happens before relation**
    - An event is something happening at one node (sending, or receiving message, or local execution step)
    - We says event $a$ happens before event $b$ (written $a \rightarrow b$) iff
        - $a$ and $b$ occurred at the same node, and $a$ occurred before $b$ in that node’s local execution order
        - Or event $a$ is sending some message $m$, and event $b$ is receiving that same mesage $m$ (assuming sent messages are unique)
        - There exists some event $c$ such that $a \rightarrow c$ and $c \rightarrow b$
        - Partial order: it is possible that $a \rightarrow b$ nor $b \rightarrow a$, in that case $a$ and $b$ are concurrent (i.e. $a || b)$
- **Causality**
    - Taking from physics (relativity)
        - When $a \rightarrow b$, then $a$ **might have caused** $b$
        - When $a || b$, we know that $a$ **cannot have caused** $b$
    - Happens before relation encodes **potential causality**

- ![alt text](images/41-communication/broadcast-hierarchy.png)
  - FIFO broadcast extends reliable boadcast with the FIFO ordering guarantee. 
  - Causal broadcast ensures that messages are delivered respecting their causal dependencies, based on the happened-before relationship defined by Lamport.
    - Since the happened-before relationship includes the order in which messages are sent by a single process, causal broadcast naturally enforces FIFO ordering for messages from the same sender.
  - Total order broadcast (also known as atomic broadcast) ensures that all messages are delivered to all processes in the same total order.
    - **This allows for multiple senders**
  - FIFO Total Order Broadcast: Combines FIFO and total order guarantees, ensuring that messages from the same sender are delivered **in order** and all messages are delivered **in the same total order** across all processes.

- Choice of retry semantics
    - **At-most-once**: send request, don’t retry, update may not happen
    - **At-least-once**: retry request until ed ACK, may repeat update
    - **Exactly-once**: retry + idempotence or deduplication

## State machine replication

- Broadcast protocol to do replication!
- Total order broadcast: every node delivers the **same messages** in **same order**
- **State machine replication (SMR)**
    - **FIFO-total order** broadcast
    - Replica delivers update message: apply it to own state
    - Applying an update is deterministic
        - Each replica is state machine, state is all data it’s stored
            - Go through same sequence of state transitions in the same order
            - Ended up in the same state

### Consensus system models

- Paxos, Raft, etc. assume a **partially synchronous, crash-recovery** system model
  - It can be designed for crash-stop, but needs additional care for state persistence and quorum adjustments. 
- Why not async?
    - **FLP result**: there is no deterministic consensus algorithm that is guaranteed to terminate in an async crash-stop system model
    - Paxos, Raft, etc. use clocks only used for timeouts / failure detector to ensure progress. Safety (correctness) does not depend on timing.

### Leader election

- Multi-paxos, Raft, etc. use a leader to sequence messages
    - Use a **failure detector** (timeout) to determine suspected crash or unavailability of leader
    - On suspected leader crash, **elect a new one**
    - Prevent **two leaders at the same time** (”split-brain”)!

## SSL / TLS: a protocol, not a software package

- Standard method of communicating between processes in modern system is socket
- Secure Socket Layer (SSL)
    - Formally is insecure
    - Move encrypted data through an ordinary socket
    - I.e. set up a socket, set up a special structure to perform crypto, then hook the output of that structure to the input of the socket, reverse the process on the other end
    - Steps
        - Start negotiation between client and server
        - End in both sides finding some acceptable **set of ciphers and techniques** that balance between performance and security
            - I.e. Diffie-Hellman key exchange to create the key
- Transport Layer Security (TLS)
    - Use this
    - Evolved from SSL.
    - It should be noted that TLS does not secure data on end systems. It simply ensures the secure delivery of data over the Internet, avoiding possible eavesdropping and/or alteration of the content.
    - TLS is normally implemented on top of TCP in order to encrypt Application Layer protocols such as HTTP, FTP, SMTP and IMAP, although it can also be implemented on UDP, DCCP and SCTP as well 
    - However, SSL is an older technology that contains some security flaws. Transport Layer Security (TLS) is the upgraded version of SSL that fixes existing SSL vulnerabilities.

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


#### Sequential Consistency
* **Order Maintenance**: The order of operations within each individual process is preserved.
* **Global Sequence**: There is a single global sequence of operations that all processes agree on, but it does not necessarily reflect the real-time order of operations.

#### Linearizability
In essence, linearizability is a more stringent requirement that includes the guarantees of sequential consistency plus the real-time ordering of operations.
```
Real-Time:   ---A1---A2---B1---B2---
Lineariz:    ---A1---A2---B1---B2---  (valid)
             ---A1---B1---A2---B2---  (valid)
             ---B1---A1---B2---A2---  (invalid, A1 < A2 in real-time)
```
#### Serializability vs. Linearizability

* **Serializability** is a global property; a property of an entire history of operations/transactions. 
* **Linearizability** is a local property; a property of a single operation/transaction. Another distinction is that linearizability includes a notion of real-time, which serializability does not

In Plain English
* Under linearizability, writes should appear to be instantaneous. Imprecisely, once a write completes, all later reads (where “later” is defined by wall-clock start time) should return the value of that write or the value of a later write. Once a read returns a particular value, all later reads should return that value or the value of a later write.
* Serializability is a guarantee about transactions, or groups of one or more operations over one or more objects. It guarantees that the execution of a set of transactions (usually containing read and write operations) over multiple items is equivalent to some serial execution (total ordering) of the transactions.