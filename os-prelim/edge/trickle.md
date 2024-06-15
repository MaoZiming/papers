# Trickle: A Self-Regulating Algorithm for Code Propagation and Maintenance in Wireless Sensor Networks (2004) 
Trickle is an algorithm designed to efficiently update code across a network of sensor nodes. It aims to save bandwidth and power by smartly deciding when a node should broadcast updates.

It particularly aims to resolve the paradox where the cost of figuring out if new code is needed overwhelms the actual cost of sending that code. The focus is thus to efficiently decide when a sensor node (mote) should propagate code.

### Techniques 
1. Randomized Timing: Each node picks a random time to announce updates. This helps avoid data collisions.
2. **Polite Gossip**: Nodes listen before they speak. If they hear enough neighbors (i.e. $k$) already talking about the same update, they keep quiet.
3. Adaptive Timing: If nothing new is happening, nodes gradually wait longer before making announcements.
4. Quick Reset: If a node hears about a newer update from a neighbor, it resets its timer to speed up the update process.

Configurable:
Two main settings: $k$, which tells us how many neighbors should be talking before a node keeps quiet, and $T$, which is our main time window for announcements.

### Limitations 
* May create redundant messages due to timing issues
* Could consume more battery since nodes are always listening.
