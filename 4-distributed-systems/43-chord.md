# Chord: A Scalable Peer-to-peer Lookup Service for Internet Applications (2001)

Link: https://pdos.csail.mit.edu/papers/chord:sigcomm01/chord_sigcomm.pdf

Read: Dec 25th, 202* 

This paper is about a new scalable distributed protocol for lookup in a **dynamic p2p system** with frequent **node arrivals and departures**. They achieve this by having a routing table with information of fewer other nodes, and consistent hashing to create identifiers for keys & nodes. 

## Baselines before consistent hashing 
Example:
* Nodes: A, B, C
* Keys: 1, 2, 3, 4, 5
* Hash Function: `hash(key)` % 3

### With consistent hashing 
Nodes and keys are hashed using a uniform hash function, placing them onto a circular hash space, or "hash ring". Keys are assigned to nodes going clockwise on the hash ring until a node is encountered.

* Nodes: A, B, C, D (hashed to values 0, 42, 84, 127 on a 0-127 hash ring)
* Keys: 1, 2, 3, 4, 5 (hashed to various positions on the same ring)
* Assigning keys based on the nearest node moving **clockwise** on the ring.

## Key Techniques 
* Identify the core operation in most p2p system: efficient location of data items
    *  Main operation: "given a key, it maps the key onto the node"  
* **Consistent hashing**: load balancing, involves little movement of keys when nodes join and leave
    *  Idea: hash both the object and the server
    *  Connect both ends of the hash to form a hash ring
    *  To locate the server for a particular object by going **clockwise**
    *  Pros: adding and removing new server only requires redistribution of a fraction of the keys
*  **Implement consistency hashing?** (to find a key location) 
    *  Option 1: every node knows location of every other node - lookup O(1), table space O(N)
    *  Option 2: every node only knows successor - Table space O(1), Lookup O(N)   
* **Finger table**: store only a few information about other nodes in each node for efficient lookup
    *  Every node knows M other nodes in the ring  
    *  Lookup efficiency in $N$ node network: $O(log N)$
    *  The $ith$ entry in the finger table at node $n$ contains the identity of the first node $s$ that succees $n$ by at least $2^i$
    *  ![Finger Table](images/43-chord/finter-table.png)

## Node changes

* In a dynamic network, nodes can **join and leave** at any time. 
* Whenever a node joins or leaves, Chord needs to preserve two invariants to ensure correctness. 
  * Each node’s successor should be correctly maintained and the second one is for every key $k$, node $successor(k)$ is responsible for $k$. 
  * In order for lookups to be fast, it is also desirable for the finger tables to be correct.
* The following three tasks should be done for a newly joined node n:
  * Initialize node $n$.
  * Notify other nodes to update their **predecessors and finger tables**
  * The new node takes over its responsible keys from its successor.
* A stablization protocol should be running periodically in the background which updates finger tables and successor pointers.

## Bounds

* In the steady-state network, each node maintains routing information for only about $O(log N)$ other nodes, and resolves all lookups via $O(log N)$ messages to other nodes.

## How does the stabilization protocol in Chord maintain the consistency of the finger table and successor lists in the presence of node joins and failures?

* Answer: The stabilization protocol in Chord ensures that each node periodically verifies and updates its successor and predecessor pointers. When a node joins, it informs its immediate predecessor and successor, allowing them to update their pointers. The stabilization process also updates the finger table by checking whether the fingers still point to the correct nodes, considering the new node joins or departures. This protocol maintains consistency and ensures that the Chord ring remains correctly connected, even as nodes join or leave.

## How does Chord handle data replication, and what challenges arise in ensuring consistency across replicas?

* Answer: Chord handles data replication by replicating data on a node and its successors. This ensures availability even if a node fails. However, challenges include maintaining consistency across replicas, particularly during node joins, departures, or failures. The system must ensure that updates are propagated correctly to all replicas, and mechanisms like versioning or consensus protocols might be needed to manage conflicts, especially in highly dynamic environments.