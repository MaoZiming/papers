# Distributed Systems

- UDP includes a checksum to detect some forms of packet corruption.

- Distributed shared memory (DSM) systems enable processes on different machines to share a large, virtual address space
  - This approach is not widely in use today for a number of reasons. The largest problem for DSM is how it handles failure.

## RPC

### Stub Generator
  - The input to such a compiler is simply the set of calls a server wishes to export to clients. Conceptually, it could be something as simple as this:

```c
interface {
  int func1(int arg1);
  int func2(int arg1, int arg2);
};
```
  - The stub generator takes an interface like this and generates a few dif- ferent pieces of code. For the client, a client stub is generated, which contains each of the functions specified in the interface; a client program wishing to use this RPC service would link with this client stub and call into it in order to make RPCs.
  - To the client, the code just appears as a function call (e.g., the client calls func1(x)); internally, the code in the client stub for func1() does this:
  - Create a message buffer. A message buffer is usually just a con- tiguous array of bytes of some size.
  - Pack the needed information into the message buffer. The process of putting all of this information into a single contiguous buffer is sometimes referred to as the marshaling of arguments or the serialization of the message.
  - Send the message to the destination RPC server.
  - Wait for the reply. Because function calls are usually synchronous, the call will wait for its completion.
  - Unpack return code and other arguments. This step is also known as unmarshaling or deserialization.
  - Return to the caller. Finally, just return from the client stub back into the client code.

### Run-time library

- Many RPC packages are built on top of unreliable com- munication layers, such as UDP.
- The run-time must also handle procedure calls with large arguments, larger than what can fit into a single packet. Some lower-level network protocols provide such sender-side fragmentation (of larger packets into a set of smaller ones) and receiver-side reassembly (of smaller parts into one larger logical whole); if not, the RPC run-time may have to implement such functionality itself.


One issue that many systems handle is that of byte ordering. As you may know, some machines store values in what is known as big endian ordering, whereas others use little endian ordering. Big endian stores bytes (say, of an integer) from most significant to least significant bits, much like Arabic numerals; little endian does the opposite. Both are equally valid ways of storing numeric information; the question here is how to communicate between machines of different endianness.

# Networked and Distributed Systems
* Remote Procedure Calls (RPCs) and their implementation
* Reliable transport (TCP vs. UDP)
* Conceptual understanding of symmetric-key encryption, public-key encryption, and cryptographic hash functions (you need to know what these primitives are and how to use them, but formal security definitions are out of scope)

## Key to distributed system: failures

How can we build a working system out of parts that don’t work correctly all the time? (e.g. machine, disk, network, software) 

- By collecting together a set of machines, we can build a system that appears to rarely fail, despite the fact that its components fail regularly
- This is the central beauty and value of distributed system
- Some other issues exists as well
    - Performance
    - Security

## Concepts Explain

[Detailed Explain: Marshaling and Unmarshaling ](https://www.notion.so/Detailed-Explain-Marshaling-and-Unmarshaling-2f692e0123d7433fa93156e669bd03e5?pvs=21)

### Remote Procedure Calls (RPCs) and their implementation

1. Problem: what abstraction of communication should we use when building a distributed system? 
    1. OS abstraction: distributed shared memory (DSM)
        1. Through virtual memory system of the OS 
        2. Not widely used today for reliable distributed system 
            1. Failure handling is hard
                1. machine fails, data structure unavailable, addr space missing, etc. 
            2. Performance: remote fetching 
    2. PL abstraction makes more sense! —> RPC 
3. Goal: create wrappers so calling a function on another machine just feels like calling a local function 
    1. **“Location transparency”**: system hides where a resource is located 
4. Basics 
    1. **Stub generator**: remove pain of packing function arguments and results into messages by automating it (i.e. protocol compiler) 
        1. Contains each of the function specified in interface
        2. Client stub 
            1. Client program using RPC would link with this can call into it 
            2. Create a message buffer, pack needed information into message buffer (marshaling), send message to destination RPC server, wait for reply, unpack return code and other arguments, return to caller 
        3. Server stub 
            1. Unpack message, call into actual function, package result, send reply 
        4. Notes 
            1. Following pointer and copy 
            2. Server regards to concurrency? —> thread pool 
                1. Threads are created when server starts
                2. When message arrives, dispatched to one of the worker threads 
                3. Main thread keeps receiving data 
    2. **Run-time library:** handle most performance and reliability issues
        1. Naming: how to locate a remove service 
            1. E.x. host name, IP address 
            2. Also mechanisms to route packets: e.g. DNS, name resolution 
        2. Which transport layer protocol to use? 
            1. TCP: very expensive! (2 additional messages are sent) 
            2. UDP: RPC needs to handle reliability (i.e. via timeout/retry) 
5. RPC steps 
    1. Client arguments to message: **marshaling / serializing** 
    2. Message to server arguments
    3. Server return value to message
    4. Message to client return value: **unmarshaling / deserializing**  
6. How it relates
    1. SunRPC / ONC RPC (1980s, basis for NFS)
    2. Now: service-oriented architecture (SOA) / micro-services 
        1. Splitting a large software application into multiple services that communicate via RPC  

### Reliable Transport: UDP v.s TCP

1. **UDP (User Datagram Protocol) - Unreliable Communication Layer** 
    1. API
        1. Reads and writes over socket API (FDs) 
        2. Messages sent from / to ports to target a process on machine 
    2. Provide minimal reliability features 
        1. Messages may be lost
        2. Messages may be reordered
        3. Messages may be duplicated
        4. Only protection: checksums to ensure data not corrupted (i.e. for integrity) 
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

### Security

- **Cryptography** achieves protection by converting data’s original bit pattern into a different bit pattern, using an algorithm called cipher
- **Symmetric key encryption** (ciphers): using a single secret key shared by all parties with rights to access the data
    - Pros: simplicity, speedy
    - Cons: key distribution problem (i.e. if someone intercepts the key during transmission, whole system compromise), doesn’t provide non-repudiation (i.e. the ability to prove that a sender is the true sender)
- **Public-key encryption**: have two different keys for cryptography, one to encrypt and one to decrypt, with one keys kept secret and the other commonly made public
    - Pros: solve the key distribution problem, secure key exchange
    - Cons: speed, complexity
- **Cryptographic hashes**: special category of hash function with important properties
    - Computationally infeasible to find two inputs that will produce the same hash value
    - Any change to input will result in unpredictable change to resulting hash value
    - Computationally infeasible to infer any properties of the input based on the hash value
    - Note: no key, no one should be able to obtain the original bit patterns from the hash
    - Then, care about data integrity?
        - Take crypto hash of the data, encrypt only that hash, send both the encrypted hash and unencrypted data to partner
        - If opponent fiddles with the data in transit, decrypt the hash and repeating the hashing operation on data, find mismatch

## System Model

For each of the three parts:

- **Network:** reliable, fair-loss (i.e. eventually get through), or arbitrary
- **Nodes:** crash-stop, crash-recovery, or Byzantine
- **Timing:** synchronous, partially synchronous, or asynchronous
    - Synchronous: message latency no greater than a known upper bound
    - Partial synchronous: async for some finite (but unknown) period of time, sync otherwise
    - Async: messages can be delayed arbitrarily, nodes can pause execution arbitrarily, no timing guarantees at all

- **Happens before relation**
    - An event is something happening at one node (sending, or receiving message, or local execution step)
    - We says event $a$  happens before event $b$ (written $a \rightarrow b$) iff
        - $a$ and $b$ occurred at the same node, and $a$ occurred before $b$ in that node’s local execution order
        - Or event $a$  is sending some message $m$, ad event $b$ is receiving that same mesage $m$ (assuming sent messages are unique)
        - There exists some event $c$ such that $a \rightarrow c$ and $c \rightarrow b$
        - Partial order: it is possible that $a \rightarrow b$ nor $b \rightarrow a$, in that case $a$ and $b$ are concurrent (i.e. $a || b)$
- **Causality**
    - Taking from physics (relativity)
        - When $a \rightarrow b$, then $a$ **might have caused** $b$
        - When $a || b$, we know that $a$ **cannot have caused** $b$
    - Happens before relation encodes **potential causality**

![alt text](image.png)

- Choice of retry semantics
    - **At-most-once**: send request, don’t retry, update may not happen
    - **At-least-once**: retry request untiled ACK, may repeat update
    - **Exactly-once**: retry + idempotence or deduplication

## State machine replication

- Broadcast protocol to do replication!
- Total order broadcast: every node delivers the **same messages** in **same order**
- **State machine replication (SMR)**
    - **FIFO-total order** broadcast every update to all replicas
    - Replica delivers update message: apply it to own state
    - Applying an update is deterministic
        - Each replica is state machine, state is all data it’s stored
            - Go through same sequence of state transitions in the same order
            - Ended up in the same state


### Consensus system models

- Paxos, Raft, etc. assume a **partially synchronous, crash-recovery** system model
- Why not async?
    - **FLP result**: there is no deterministic consensus algorithm that is guaranteed to terminate in an async crash-stop system model
    - Paxos, Raft, etc. use clocks only used for timeouts / failure detector to ensure progress. Safety (correctness) does not depend on timing.
- There are also consensus algorithms for a partially synchronous Byzantine system model (used in blockchains)
    - More complex! Less efficient

### Leader election

- Multi-paxos, Raft, etc. use a leader to sequence messages
    - Use a **failure detector** (timeout) to determine suspected crash or unavailability of leader
    - On suspected leader crash, **elect a new one**
    - Prevent **two leaders at the same time** (”split-brain”)!

Distributed systems consist of multiple computers that communicate over a network to collaboratively achieve tasks. These systems can be inherently distributed (such as IoT devices), designed for better reliability, improved performance, or to tackle problems that are too large for a single machine (e.g., big data processing).

## Challenges: fault tolerance 
Faults are an everyday concern in distributed systems:
* Communication may fail (e.g. message loss) 
* Processes may crash 
* All of these may happen nondeterministically 

A _"failure"_ refers to the entire system not working, while a _"fault"_ implies that some parts of the system are malfunctioning. The goal is to achieve high availability via **fault tolerance**, meaning the system should continue to function despite experiencing faults.

## System Model 
### Network 
1. Reliable: Assumes all messages will be successfully delivered.
2. Fair-loss: Assumes messages may be lost but will eventually get through.
3. Arbitrary: No assumptions about message delivery.
### Nodes 
1. Crash-stop: Nodes will stop and not recover.
2. Crash-recovery: Nodes can recover after a crash.
3. Byzantine: Nodes can act arbitrarily, including maliciously.
### Timing 
1. Synchronous: Assumes that message latency is bounded and known.
2. Partially Synchronous: Asynchronous for some time but eventually becomes synchronous.
3. Asynchronous: No timing assumptions; messages can be delayed indefinitely, and nodes can pause arbitrarily.  

## Overview 
In the realm of distributed systems, timing, fault tolerance, and data consistency are crucial factors. 

The need to order events in a distributed system is imperative for consistency and coordination among different nodes. Lamport clock provides a way to define event orderings in a distributed system, which is foundational for achieving data consistency and system coordination. 

Building upon Lamport clock, the focus shifts to ensuring data consistency between data replicas. VR and pBFT are two replication protocol that enable systems to continue execute despite faults. VR deals with the crash-stop or crash-recovery model, while pBFT deals with Byzantine faults. 

While Lamport clocks operate primarily under synchronous assumptions, both VR and pBFT extend these ideas to systems that can function under various timing models, including partial asynchrony.

- Symmetric crypto: used for transport of most data, since it is cheaper
    - Not shared by system participants before the communication starts
    - First step in protocol: exchange symmetric keys
    - Diffie-Hellman key exchange is commonly used

## SSL / TLS: a protocol, not a software package

- Standard method of communicating between processes in modern system is socket
- Secure Socket Layer (SSL)
    - Formally is insecure
    - Move encrypted data through an ordinary socket
    - I.e. set up a socket, set up a special structure to perform crypto, then hook the output of that structure to the input of the socket, reverse the process on the other end
    - Steps
        - Start negotiation between client and server
        - End in both sides finding some acceptable set of ciphers and techniques that balance between performance and security
            - I.e. Diffie-Hellman key exchange to create the key
- Transport Layer Security (TLS)
    - Use this
    - Evolved from SSL.
    - It should be noted that TLS does not secure data on end systems. It simply ensures the secure delivery of data over the Internet, avoiding possible eavesdropping and/or alteration of the content.
    - TLS is normally implemented on top of TCP in order to encrypt Application Layer protocols such as HTTP, FTP, SMTP and IMAP, although it can also be implemented on UDP, DCCP and SCTP as well 
    - However, SSL is an older technology that contains some security flaws. Transport Layer Security (TLS) is the upgraded version of SSL that fixes existing SSL vulnerabilities.