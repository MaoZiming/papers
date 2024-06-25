# Epidemic Routing for Partially-Connected Ad Hoc Network (2000) 
The paper proposes Epidemic Routing, a routing protocol designed for ad hoc networks where connectivity is intermittent.

## Insights

- E.x. mobile sensor networks, nodes can be spread over wide geographical distances, it is unlikely that a connected path can always be discovered
- Goals: maximize message delivery rate, minimize message latency while also minimizing total resources (e.g. memory and network bandwidth) consumed in message delivery

## Techniques
- Each host maintains 
    - a buffer of messages (i.e. hash table)
    - a summary vector: which entries in their local hash tables are set
- Two-host communication
    - Host with the smaller identifier initiates an anti-entropy session
    - During anti-entropy session, two hosts exchange summary vectors to determine which messages stored remotely have not been seen
    - Then each host requires copies of messages that it has not yet seen
- Each message: a unique message identifier, a hop count, and an optional ack request
    - Hop: determines max # of epidemic exchanges that a particular message is subject to
    - Also limit per-node buffer space
      
## Limitations
- this protocol inherently relies on the randomness of node meetings to gradually spread messages through the network, it is non-deterministic and can potentially have high latency for message delivery 
