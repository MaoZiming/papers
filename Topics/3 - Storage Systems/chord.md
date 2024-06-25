# Chord: A Scalable Peer-to-peer Lookup Service for Internet Applications (2001) 
This paper is about a new scalable distributed protocol for lookup in a dynamic p2p system with frequent node arrivals and departures. They achieve this by having a routing table with information of fewer other nodes, and consistent hashing to create identifiers for keys & nodes. 

**Key**: performance, simplicity 

## Baselines before consistent hashing 
Example:
* Nodes: A, B, C
* Keys: 1, 2, 3, 4, 5
* Hash Function: hash(key) % 3

### With consistent hashing 
Nodes and keys are hashed using a uniform hash function, placing them onto a circular hash space, or "hash ring". Keys are assigned to nodes going clockwise on the hash ring until a node is encountered.

* Nodes: A, B, C, D (hashed to values 0, 42, 84, 127 on a 0-127 hash ring)
* Keys: 1, 2, 3, 4, 5 (hashed to various positions on the same ring)
* Assigning keys based on the nearest node moving clockwise on the ring would look like this:

Keys close to 0 would be assigned to A. Keys close to 42 would be assigned to B. So on and so forth.

## Key Techniques 
* Identify the core operation in most p2p system: efficient location of data items
    *  Main operation: "given a key, it maps the key onto the node"  
* **Consistent hashing**: load balancing, involves little movement of keys when nodes join and leave
    *  Idea: hash both the object and the server
    *  Connect both ends of the hash to form a hash ring
    *  To locate the server for a particular object by going clockwise
    *  Pros: adding and removing new server only requires redistribution of a fraction of the keys
*  **Implement consistency hashing?** (to find a key location) 
    *  Option 1: every node knows location of every other node - lookup O(1), tables O(N)
    *  Option 2: every node only knows successor - Table O(1), Lookup O(N)   
* **Finger table**: store only a few information about other nodes in each node for efficient lookup
    *  Every node knows M other nodes in the ring  
    *  Lookup efficiency in $N$ node network: $O(log N)$
    *  The $ith$ entry in the finger table at node $n$ contains the identity of the first node $s$ that succees $n$ by at least $2^i$

### Example w/ finger table 
With a finger table, instead of only knowing its successor, Node A might also know about Node C. So when looking for key 12, instead of having to ask each node in turn, it can skip directly to a node that it knows is closer to the key's location. In the optimal configuration, this can reduce lookup time to $O(logN)$, striking a balance between the maintenance overhead of option 1 and the slow lookups of option 2.
