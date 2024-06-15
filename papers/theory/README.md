# Distributed System Theory 
Distributed systems consist of multiple computers that communicate over a network to collaboratively achieve tasks. These systems can be inherently distributed (such as IoT devices), designed for better reliability, improved performance, or to tackle problems that are too large for a single machine (e.g., big data processing).

## Challenges: fault tolerance 
Faults are an everyday concern in distributed systems:
* Communication may fail (e.g. message loss) 
* Processes may crash 
* All of these may happen nondeterministically 

A _"failure"_ refers to the entire system not working, while a _"fault"_ implies that some parts of the system are malfunctioning. The goal is to achieve high availability via **fault tolerance**, meaning the system should continue to function despite experiencing faults.

## System Model 
### Network 
1. Reliable: Assumes all messages will be successfully delivered.
2. Fair-loss: Assumes messages may be lost but will eventually get through.
3. Arbitrary: No assumptions about message delivery.
### Nodes 
1. Crash-stop: Nodes will stop and not recover.
2. Crash-recovery: Nodes can recover after a crash.
3. Byzantine: Nodes can act arbitrarily, including maliciously.
### Timing 
1. Synchronous: Assumes that message latency is bounded and known.
2. Partially Synchronous: Asynchronous for some time but eventually becomes synchronous.
3. Asynchronous: No timing assumptions; messages can be delayed indefinitely, and nodes can pause arbitrarily.  

## Overview 
In the realm of distributed systems, timing, fault tolerance, and data consistency are crucial factors. 

The need to order events in a distributed system is imperative for consistency and coordination among different nodes. Lamport clock provides a way to define event orderings in a distributed system, which is foundational for achieving data consistency and system coordination. 

Building upon Lamport clock, the focus shifts to ensuring data consistency between data replicas. VR and pBFT are two replication protocol that enable systems to continue execute despite faults. VR deals with the crash-stop or crash-recovery model, while pBFT deals with Byzantine faults. 

While Lamport clocks operate primarily under synchronous assumptions, both VR and pBFT extend these ideas to systems that can function under various timing models, including partial asynchrony.
