# Epidemic Routing for Partially-Connected Ad Hoc Network (2000) 

Link: http://issg.cs.duke.edu/epidemic/epidemic.pdf

Read: June 28th, 2024

* The paper proposes Epidemic Routing, a routing protocol designed for ad hoc networks where connectivity is intermittent.

* The goals of **Epidemic Routing**: 
  * Maximize message delivery rate
  * Minimize message latency
  * Minimize the total resources consumed in message delivery.

* The goal is to develop techniques for delivering application data with high probability even when there is never a fully connected path between source and destination.   
  * Assumptions about the connectivity of the underlying ad hoc network:
  * The sender is never in range of any base stations
  * The sender does not know where the receiver is currently located or the best “route” to follow
  * The receiver may also be a roaming wireless host.
  * Pairs of hosts (not necessarily the sender and receiver) periodically and randomly come into communication range of one another through node mobility.

![alt text](images/81-epidemic-routing/diagram.png)

## Design

* Each host maintains a buffer consisting of messages that it is buffering on behalf of other hosts. Each message is associated with age and a unique identifier (hash). 

* Each host summary vector that indicates which entries in their local hash tables are set. Bloom filter would substantially reduce the space overhead associated. 

* During **anti-entropy**, the two hosts exchange their summary vectors to determine which messages stored remotely have not been seen by the local host. In turn, each host then requests copies of messages that it has not yet seen. Each host may decide whether it can be the carrier for a particular message.

* Given that messages are delivered probabilistically in epidemic routing, certain applications may require acknowledgments of message delivery. The ack request field signals the destination of a message to provide an acknowledgment of message delivery. These acknowledgments are modeled as simple return messages from receiver back to the sender. Of course, the acknowledgment can also be piggybacked with any other message destined back to the sender after the message is successfully delivered. As future work, we intend to experiment with supplementing **anti-entropy** with the exchange of a “message delivered” vector. This vector can act as both both message acknowledgment and as a capability to free the buffer space associated with messages that have been previously delivered.

* Each host sets a maximum buffer size that it is willing to allocate for epidemic message distribution. The buffer size limits the amount of memory and network resources consumed through Epidemic Routing.

## Techniques
* Each host maintains 
    * a buffer of messages (i.e. hash table)
    * a summary vector: which entries in their local hash tables are set
* Two-host communication
    * Host with the smaller identifier initiates an anti-entropy session
    * During anti-entropy session, two hosts exchange summary vectors to determine which messages stored remotely have not been seen
    * Then each host requires copies of messages that it has not yet seen
* Each message: a unique message identifier, a hop count, and an optional ack request
    * Hop: determines max # of epidemic exchanges that a particular message is subject to
      * Assign a larger hop count for more important messages. 
    * Also limit per-node buffer space
      
## Limitations
* this protocol inherently relies on the randomness of node meetings to gradually spread messages through the network, it is non-deterministic and can potentially have high latency for message delivery 
