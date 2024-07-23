# Paxos Made Simple

Link: https://lamport.azurewebsites.net/pubs/paxos-simple.pdf

Read: July 23rd, 2024

I don't understand why Prelim doesn't require Paxos!

## Safety

* Only a value that has been proposed may be chosen
* Only a single value is chosen, and
* A process never learns that a value has been chosen unless it actually has been.

## Agents

* A process can be mapped to >=1 agents.
* Three classes: proposers, acceptors, learners.

## Network

* Asynchronous network
* Non-Byzantine failures

## Proposal

* Proposal number and value.
* A value is chosen when a single proposal with that value has been accepted by a majority of the acceptors.

## Phase 1

### (a)
* A proposer chooses a new proposal number $n$ and sends a request to each member of some set of acceptors, asking it to respond with:
  * A promise never again to accept a proposal numbered less than $n$
  * The proposal with the highest number less than $n$ that it has accepted, if any.
I will call such a request a prepare request with number $n$.

### (b)

* If an acceptor receives a prepare request with number $n$ greater than that of any prepare request to which it has already responded, then it responds to the request with a promise not to accept any more proposals numbered less than $n$ and with the highest-numbered proposal (if any) that it has accepted.

## Phase 2

### (a)
* If the proposer receives the requested responses from a majority of the acceptors, then it can issue a accept request with number $n$ and value $v$, where $v$ is the value of the highest-numbered proposal among the responses, or is any value selected by the proposer if the responders reported no proposals.

### (b)
* If an acceptor receives an accept request for a proposal numbered n, it accepts the proposal unless it has already responded to a prepare request having a number greater than n.

## Accept

> An acceptor needs to remember only the highest- numbered proposal that it has ever accepted and the number of the highest- numbered prepare request to which it has responded.

## Learners

We can have the acceptors respond with their acceptances to a distinguished learner, which in turn informs the other learners when a value has been chosen.
* The distinguished learner can fail.
* But leads to fewer messages!

## Distinguished Proposer

To guarantee progress, a distinguished proposer must be selected as the only one to try issuing proposals.

> In its consensus algorithm, each process plays the role of proposer, acceptor, and learner. The algorithm chooses a leader, which plays the roles of the distinguished proposer and the distinguished learner.