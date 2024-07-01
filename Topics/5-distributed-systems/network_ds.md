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
