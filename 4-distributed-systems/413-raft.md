# In Search of an Understandable Consensus Algorithm

Link: https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14.pdf

Read: July 12th, 2024

> Our approach was unusual in that our primary goal was understandability: could we define a consensus algorithm for practical systems and describe it in a way that is significantly easier to learn than Paxos? 

## Novel features

* Strong leader: log entries only flow from the leader to other servers. 
* Leader election: Raft uses randomized timers to elect leaders.
* Membership changes: new joint consensus approach where the majorities of two different configurations overlap during transitions.

## Problems with Paxos

* Choice of the single-decree subset as its foundation.
  * divided into two stages that do not have simple intuitive explanations and cannot be understood independently. 
* Does not provide a good foundation for building practical systems.
* Symmetric p2p approach. It is simpler and faster to elect a leader, and have the leader coordinate the decisions. 

## Raft algorithm

* Terms:
  * Raft divides time into terms of arbitrary length.
  * Terms are numbered with consecutive integers. Each term begins with an election, in which one or more candidates attempt to become leader as described in
* In some situations an election will result in a split vote. In this case the term will end with no leader; a new term (with a new election) will begin shortly. 
* Raft servers communicate using remote procedure calls (RPCs), 

* Log replication:
  * Once a leader has been elected, it begins servicing client requests. Each client request contains a command to be executed by the replicated state machines. The leader appends the command to its log as a new entry, then issues AppendEntries RPCs in parallel to each of the other servers to replicate the entry. 
* Inconsistency resolution:
  * In Raft, the leader handles inconsistencies by forcing the followers’ logs to duplicate its own. This means that conflicting entries in follower logs will be overwritten with entries from the leader’s log.
    * A leader never overwrites or deletes entries in its own log
* Leader committing an entry:
  * Raft never commits log entries from previous terms by counting replicas. Only log entries from the leader’s current term are committed by counting replicas; once an entry from the current term has been committed in this way, then all prior entries are committed indirectly because of the Log Matching Property.