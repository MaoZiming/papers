# Mesos: A Platform for Fine-Grained Resource Sharing in the Data Center

Link: https://people.eecs.berkeley.edu/~alig/papers/mesos.pdf

* It seems clear that new cluster computing frameworks will continue to emerge, and that no framework will be optimal for all applications. Therefore, organizations will want to run multiple frameworks in the same cluster, picking the best one for each application.
  * Multiplexing a cluster between frameworks

## Summary

* Enables fine-grained sharing across diverse cluster computing frameworks, by giving frameworks a common interface for accessing cluster resources.

## Challenges

* Each framework will have different scheduling needs, based on its programming model, commuication patterns, task dependencies and data placements.
* The scheduling system must scale to tens of thousands of nodes and support millions of tasks.
* All applications depend on Mesos, the system must be fault-tolerant and highly available. 

## Design

* Delegating control over scheduling to the frameworks.
* Resource offer. 
* Mesos decides how many resources to offer each framework, based on an organizational policy such as fair sharing, while frameworks decide which resources to accept and which tasks to run on them.
* decentralized scheduling model may not always lead to globally optimal scheduling, we have found that it performs sur- prisingly well in practice, allowing frameworks to meet goals such as data locality nearly perfectly.