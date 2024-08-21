# Time, Clocks, and the Ordering of Events in a Distributed System

Link: https://lamport.azurewebsites.net/pubs/time-clocks.pdf

Read: Jan 23th, 202* 

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
* We only have **partial order** in the system
* Definition
    * In the same process, if event $a$ comes before event $b$, then $a \rightarrow b$
    * When process $i$ sends some message at $a$ to process $j$ and process $j$ ack this message at $b$, then $a \rightarrow b$
    * Transitive: if $a \rightarrow b$ and $b \rightarrow c$, then $a \rightarrow c$
    * Event $a$  and event $b$ are concurrent when $a$ hasn’t happened before $b$ and $b$ hasn’t happened before $a$

Lamport introduces **logical clock**, which is the way to count number of events that have occurred.

* $a \rightarrow b$  implies $C(a) < C(b)$
* **Partial ordering** is defined
    * $C_i(a) < C_i(b)$ if $a$ happens before $b$ in the same process $i$. This can be implemented using a **simple counter** in the given process.
    * When process $i$ sends message at event $a$ and process $j$ ack the message at event $b$, then $C_i(a) < C_j(b)$
  
### Synchronization Algorithm 
The paper offers a simple algorithm for advancing the logical clocks in a way that respects the happened-before relationship.
* Each node maintains **a counter $t$**, incremented on local event $e$
* When the node sends the message, attach current $t$ to messages sent over the network
* Recipients move its clock forward to timestamp in the message (if greater than the local counter), then increments 


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
* **Vector Clock**: A vector clock for a system with $N$ processes is an array of $N$ integers.
* **Each Process**: Each process $P_i$ maintains its own vector clock $VC_i$, where $VC_i[j]$ is the process $P_i$'s knowledge of the logical time at process $P_j$.

### Working with Vector Clocks
* **Initialization**: Each process $P_i$ initializes its vector clock to an array of zeros, with length equal to the number of processes in the system:
   * $VC_i[k] = 0$ for all $k$.

* **Event Occurrence**: When a process $P_i$ performs an event (e.g., sending a message or performing a local operation), it increments its own entry in its vector clock:
   * $VC_i[i] = VC_i[i] + 1$.

* **Message Sending**: When process $P_i$ sends a message to process $P_j$, it includes a copy of its vector clock $VC_i$ with the message.

* **Message Receiving**: When process $P_j$ receives a message from process $P_i$ with the vector clock $VC_i$:
   * It increments its own entry: $VC_j[j] = VC_j[j] + 1$.
   * For each other entry $k$, it updates its vector clock to the maximum value of its own clock and the received clock: $VC_j[k] = \max(VC_j[k], VC_i[k])$.

## Vector clock vs. logical clock

* **Logical Clocks**: Simple, single counter per process, provides a total order (can be extended with tie breaking. ) but limited causal information.
* **Vector Clocks**: More complex, vector of counters per process, provides detailed causal ordering and concurrency detection.
  * **Causal ordering**: Event $A$ causally precedes event $B$ if $VC_A \leq VC_B$ (i.e., for all $k$, $VC_A[k] \leq VC_B[k]$) and $VC_A \neq VC_B$.
  * **Concurrency Detection**: Two events are concurrent if their vector clocks are incomparable. 

## Mutual Exclusion example

* Three type of messages ( REQUEST, REPLY and RELEASE) are used and communication channels are assumed to follow FIFO order.
  * A site send a REQUEST message to all other site to get their permission to enter critical section.
  * A site send a REPLY message to requesting site to give its permission to enter the critical section.
  * A site send a RELEASE message to all other site upon exiting the critical section.
* Every site Si, keeps a queue to store critical section requests ordered by their timestamps. request_queuei denotes the queue of site Si
* A timestamp is given to each critical section request using Lamport’s logical clock.
* Timestamp is used to determine priority of critical section requests. Smaller timestamp gets high priority over larger timestamp. The execution of critical section request is always in the order of their timestamp.

* To enter Critical section:
* When a site Si wants to enter the critical section, it sends a request message Request(tsi, i) to all other sites and places the request on request_queuei. Here, Tsi denotes the timestamp of Site Si
* When a site Sj receives the request message REQUEST(tsi, i) from site Si, it returns a timestamped REPLY message to site Si and places the request of site Si on request_queuej
* To execute the critical section:
* A site Si can enter the critical section if it has received the message `REPLY` with timestamp larger than (tsi, i) from all other sites and its own request is at the top of request_queuei
* To release the critical section:
* When a site Si exits the critical section, it removes its own request from the top of its request queue and sends a timestamped RELEASE message to all other sites
* When a site Sj receives the timestamped RELEASE message from site Si, it removes the request of Si from its request queue

### Drawbacks

* Unreliable approach: failure of any one of the processes will halt the progresse of the entire system.
* High message complexity: Algorithm requires $3(N-1)$ messages for each critical section request.
  * $(N-1)$ messages for REQUEST, $(N-1)$ messages for REPLY and $(N-1)$ messages for RELEASE.