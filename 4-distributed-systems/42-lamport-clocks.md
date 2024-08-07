# Time, Clocks, and the Ordering of Events in a Distributed System

Link: https://lamport.azurewebsites.net/pubs/time-clocks.pdf

Read: Jan 23th, 2024. 

## Motivation: Event Ordering 
* Event ordering is crucial for various aspects of system functionalities such as data consistency, debugging, and coordination.
* There is no single, universally-agreed-upon time or clock that all processes can refer to.
* Synchronizing physical clocks cannot solve the problem due to issues like clock drift and network latency. 
* Logical clock includes {Lamport clock, vector clock, matrix clocks}. 
  * Both provides total ordering of events consistent with causality
  * Vector clocks allow you to determine if any two arbitrarily selected events are causally dependent or concurrent. Lamport timestamps cannot do this.

## Summary 
Lamport presents the classic paper about how to define ordering of events in system of collections of nodes. 

An event is something happening at one node (i.e. sending or receiving messsage, or local execution step). The key idea about this paper is the concept about **"happened-before" relationship**, which gives us a way to compare two events from potentially different processes.
- We only have **partial order** in the system
- Definition
    - In the same process, if event $a$ comes before event $b$, then $a \rightarrow b$
    - When process $i$ sends some message at $a$ to process $j$ and process $j$ ack this message at $b$, then $a \rightarrow b$
    - Transitive: if $a \rightarrow b$ and $b \rightarrow c$, then $a \rightarrow c$
    - Event $a$  and event $b$ are concurrent when $a$ hasn’t happened before $b$ and $b$ hasn’t happened before $a$

Lamport introduces **logical clock**, which is the way to count number of events that have occurred.

- $a \rightarrow b$  implies $C(a) < C(b)$
- **Partial ordering** is defined
    - $C_i(a) < C_i(b)$ if $a$ happens before $b$ in the same process $i$. This can be implemented using a simple counter in the given process.
    - When process $i$ sends message at event $a$ and process $j$ ack the message at event $b$, then $C_i(a) < C_j(b)$
  
### Synchronization Algorithm 
The paper offers a simple algorithm for advancing the logical clocks in a way that respects the happened-before relationship.
1. Each node maintains **a counter $t$**, incremented on local event $e$
2. When the node sends the message, attach current $t$ to messages sent over the network
3. Recipients move its clock forward to timestamp in the message (if greater than the local counter), then increments 


**Partial ordering** with lamport clock is useful to establish causation of messages. To produce a **total ordering**, Lamport introduces the notion of tie-breaking based on the deterministic ordering of processes. 
* The tie breaker can be an unique node ID. 
* Because the node ID can be arbitrarily used to break ties, we effectively have a total order. In other words, we can "convert" a partially ordered sequence number into a totally ordered one that is consistent with causality. It's still not a linearizable system, which makes it feel partially ordered.

## Limitations 
* This paper does not present any mechanisms for failure. 
* Also, given Lamport timestamps $L(a)$ and $L(b)$, with $L(a) < L(b)$, we can't tell whether $a \rightarrow b$ or $a || b$. We can tell something, but can't differentiate the event that is concurrent and the event that one happened before the other. If we want to detect which events are concurrent, we need vector clocks.  

In summary, the limitations are:
* **Partial ordering: can't determine order of concurrent events**
* Doesn't handle network failures: delays or message losses can lead to potential inconsistency
* Overhead: carry timestamp information

Scenarios where they are insufficient:
* Synchronization of concurrent events: google doc, collaborative editing, gaming 
* Global total ordering requirements: i.e. tradings 
* Timestamp transactions: i.e. financial market, where microsecond precision is required for transaction ordering, relying on logical clock can introduce anomalies 

## Vector clock

### Structure of a Vector Clock
- **Vector Clock**: A vector clock for a system with $N$ processes is an array of $N$ integers.
- **Each Process**: Each process $P_i$ maintains its own vector clock $VC_i$, where $VC_i[j]$ is the process $P_i$'s knowledge of the logical time at process $P_j$.

### Working with Vector Clocks
1. **Initialization**: Each process $P_i$ initializes its vector clock to an array of zeros, with length equal to the number of processes in the system:
   - $VC_i[k] = 0$ for all $k$.

2. **Event Occurrence**: When a process $P_i$ performs an event (e.g., sending a message or performing a local operation), it increments its own entry in its vector clock:
   - $VC_i[i] = VC_i[i] + 1$.

3. **Message Sending**: When process $P_i$ sends a message to process $P_j$, it includes a copy of its vector clock $VC_i$ with the message.

4. **Message Receiving**: When process $P_j$ receives a message from process $P_i$ with the vector clock $VC_i$:
   - It increments its own entry: $VC_j[j] = VC_j[j] + 1$.
   - For each other entry $k$, it updates its vector clock to the maximum value of its own clock and the received clock: $VC_j[k] = \max(VC_j[k], VC_i[k])$.

## Vector clock vs. logical clock

- **Logical Clocks**: Simple, single counter per process, provides a total order but limited causal information.
- **Vector Clocks**: More complex, vector of counters per process, provides detailed causal ordering and concurrency detection.
  -  **Causal ordering**: Event $A$ causally precedes event $B$ if $VC_A \leq VC_B$ (i.e., for all $k$, $VC_A[k] \leq VC_B[k]$) and $VC_A \neq VC_B$.
  -  **Concurrency Detection**: Two events are concurrent if their vector clocks are incomparable. 
