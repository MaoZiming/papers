# Trickle: A Self-Regulating Algorithm for Code Propagation and Maintenance in Wireless Sensor Networks (2004) 

Link: http://csl.stanford.edu/~pal/pubs/trickle-nsdi04.pdf

Read: June 28th, 2024,

* Trickle is an algorithm designed to efficiently update code across a network of sensor nodes. It aims to save bandwidth and power by smartly deciding when a node should broadcast updates.

* A large number of small, resource constrained computing nodes ("motes"). Motes come and go, due to temporary disconnections, failure, and network repopulation.

* To reduce energy costs, motes can transmit metadata to determine when code is needed. Even for binary images, this periodic metadata exchange overwhelms the cost of transmitting code when it is needed. Sending a full TinyDB binary image (≈ 64 KB) costs approximately the same as transmitting a forty byte metadata summary once a minute for a day. 

  * This is an important property in wireless networks, where the channel is a valuable shared resource. Additionally, reducing transmissions in dense networks conserves system energy. 

## Insights

- Sending a few metadata packets costs the same as sending the entire program
- The communication to learn when code is needed overwhelms the cost of actually propagating that code
- Trickle aims to efficiently determine when motes should propagate code

### Techniques 
1. Randomized Timing: Each node picks a random time to announce updates. This helps avoid data collisions.
2. **Polite Gossip**: Nodes listen before they speak. If they hear enough neighbors (i.e. $k$) already talking about the same update, they keep quiet.
3. Adaptive Timing: If nothing new is happening, nodes gradually wait longer before making announcements.
4. Quick Reset: If a node hears about a newer update from a neighbor, it resets its timer to speed up the update process.
5. Silent listening period before gossiping. 

6. Configurable:
Two main settings: $k$, which tells us how many neighbors should be talking before a node keeps quiet, and $T$, which is our main time window for announcements.

### Limitations 
* May create redundant messages due to timing issues
* **Could consume more battery since nodes are always listening**.

- It exchanges metadata in a gossip way without requiring to know the global information
    - “polite gossip”: reduce medium contention and power usage
    - A mote will listen to its neighbors before it announces its own message
    - If a mote or its neighbors are out of date then the mote will announce its metadata in the hopes of being updated
    - If a mote hears enough of its neighbors announcing same metadata, it will wait until the next time window to announce
    - If it doesn’t hear enough, it will announce its own metadata

- Configurable parameter
    - $k$: the neighborhood comm threshold that decides if a mote should announce its metadata
    - $T$: length of the trickle broadcast window
        - Im each interval $T$, each mote randomly picks a time $t$ within $T$, and broadcast metadata at time $t$ unless it hears at least $k$ other nodes broadcasting same metadata before time $t$
        - Increase $T$ if there’s no new code
        - Resets $T$ if there is

## Limitations

- Practical environment: add extra redundant messages overhead. 
    - Without synchronization: short-listen problem
    - Multi-hop network: hidden terminal problem
- All motes need to listen to broadcasts from neighbors continuously, might be power-consuming