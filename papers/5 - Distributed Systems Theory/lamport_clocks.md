# Time, Clocks, and the Ordering of Events in a Distributed System

## Motivation: Event Ordering 
* Event ordering is crucial for various aspects of system functionalities such as data consistency, debugging, and coordination.
* There is no single, universally-agreed-upon time or clock that all processes can refer to.
* Synchronizing physical clocks cannot solve the problem due to issues like clock drift and network latency. 

## Summary 
Lamport presents the classic paper about how to define ordering of events in system of colleciton of nodes. 

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
1. Each node maintains a counter $t$, incremented on local event $e$
2. When the node sends the message, attach current $t$ to messages sent over the network
3. Recipients move its clock forward to timestamp in the message (if greater than the local counter), then increments 
<img width="567" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/7c3c8436-f802-4ad7-95d5-9b59811884db">


**Partial ordering** with lamport clock is useful to establish causation of messages. To produce a **total ordering**, Lamport introduces the notion of tie-breaking based on the deterministic ordering of processes. 

## Examples 
A, B, C: A broadcast questions, B replies based on that question 

## Limitations 
This paper does not present any mechanisms for failure. Also, given Lamport timestamps $L(a)$ and $L(b)$, with $L(a) < L(b)$, we can't tell whether $a \rightarrow b$ or $a || b$. We can tell something, but can't differentiate the event that is concurrent and the event that one happened before the other. If we want to detect which events are concurrent, we need vector clocks.  

In summary, the limitations are:
* Partial ordering: can't determine order of concurrent events
* Doesn't handle network failures: delays or message losses can lead to potential inconsistency
* Overhead: carry timestamp information

Scenarios where they are insufficient:
* Synchronization of concurrent events: google doc, collaborative editing, gaming 
* Global total ordering requirements: i.e. tradings 
* Timestamp transactions: i.e. financial market, where microsecond precision is required for transaction ordering, relying on logical clock can introduce anomalies 
