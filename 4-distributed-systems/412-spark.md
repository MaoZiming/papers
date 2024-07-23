# Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing

Link: https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf

Read: April 14th, 2024

* Those that reuse intermediate results across multiple computations. Data reuse is common in many iterative machine learning and graph algorithm. 
  * Unfortunately, in most current frameworks, the only way to reuse data between computations (e.g., between two MapReduce jobs) is to write it to an external stable storage system, e.g., a distributed file system.
* Resilient Distributed Datasets (RDDs), a distributed memory abstraction that lets programmers perform in-memory computations on large clusters in a fault-tolerant manner.
* RDDs are motivated by two types of applications that current computing frameworks handle inefficiently: iterative algorithms and interactive data mining tools.
* RDDs provide an interface based on coarse-grained transformations (e.g., map, filter and join) that apply the same operation to many data items. This allows them to efficiently provide fault tolerance by logging the transformations used to build a dataset (its lineage) rather than the actual data.

## RDD

* RDD is read-only, can be derived from other RDDs or data on stable storage.
* Transformations: `map` , `filter`, and `join`. 
* RDD has enough information about how it was derived from other datasets (**lineage**)
* The user can specify the storage medium for the RDD (memory or disk), and partitioning across machines given a key.

## Comparison to DSM

* DSM is general: applications read and write to arbitrary locations in a global address space. 
  * Complicates fault tolerance.
* RDDs can only be created with coarse-grained transformations.
  * Do not incur the overhead of checkpointing as RDD can be recovered with lineage. 
  * RDD can be recomputed in parallel on different nodes. 
  * Mitigate slow nodes by running backup copies on different machines. 

## Applications not suitable for RDDs

* RDDs would be less suitable for applications that make **asynchronous** updates to shared states, such as storage system for web application or incremental web crawler. 

## System architecture

* A driver program connected to a cluster of workers.
* ![alt text](images/412-spark/spark-runtime.png)
* RDD themselves are statically typed objects. 

## Narrow vs. Wide Dependencies
* Narrow dependencies: each partition of the child RDD depends on a small number of partitions of the parent RDD.
* Wide dependencies: each partition of the child RDD depends on multiple partitions of the parent RDD. This requires data to be shuffled across the nodes.
* Narrow dependencies allow for pipelined execution on one cluster node. 
  * e.g. One can apply a *map* followed by a *filter* on an element-by-element basis. 
  * In contrast, wide dependencies require data from all parent partitions to be available and to be shuffled across the nodes using a MapReduce-like operation.
* Second, recovery after a node failure is more efficient with a narrow dependency, as only the lost parent partitions need to be recomputed, and they can be recomputed in parallel on different nodes. In contrast, in a lineage graph with wide dependencies, a single failed node might cause the loss of some partition from all the ancestors of an RDD, requiring a complete re-execution.
* ![alt text](images/412-spark/narrow-vs-wide.png)