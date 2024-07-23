# Parameter Server

Link: https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-li_mu.pdf

Read: July 3rd 2024. 

## Challenges

* Accessing the parameters requires an enormous amount of network bandwidth.
* Many machine learning algorithms are sequential. The resulting barriers hurt performance when the cost of synchronization and machine latency is high.
* At scale, fault tolerance is critical. Learning tasks are often performed in a cloud environment where machines can be unreliable and jobs can be preempted

## Designs

* Efficient communication: The asynchronous communication model does not block computation (unless requested).
* Flexible consistency models: Relaxed consistency fur- ther hides synchronization cost and latency.
* Elastic Scalability: New nodes can be added without restarting the running framework.
* Fault Tolerance and Durability: Recovery from and repair of non-catastrophic machine failures within 1s, with- out interrupting computation. 
* Ease of Use: The globally shared parameters are represented as (potentially sparse) vectors and matrices to facil- itate development of machine learning applications. 

![alt text](images/46-parameter_server/architecture.png)

## Architecture

* Server Group
  * A server node in the server group maintains a partition of the globally shared parameters.
  * A server manager node maintains a consistent view of the metadata of the servers, such as node liveness and the as- signment of parameter partitions.
  * A server manager node maintains a consistent view of the metadata of the servers, such as node liveness and the assignment of parameter partitions.
* Worker group
  * Each worker group runs an application. 
  * A worker stores locally a portion of the training data to communicate local statistics such as gradients. 
  * Worker only communicates with server nodes (not among themselves), updating and retrieivng shared parameters. 

## Range push and pull

* Data is sent between nodes using push and pull operations. 
  * In one algorithm, each worker pushes its entire local gradient into the servers, and then pulls the updated weight back. 
  * The more advanced algorithm uses the same pattern, except that only a range of keys is communicated each time.

## UDF

* The server node can execute user-defined functions. 
  * E.g. server nodes evaluate subgradients of the regularizer $Ω$ in order to update $w$.

## Strong Points

- The trade-off between the system efficiency and algorithm convergence rate depends on: algorithm’s sensitivity to data inconsistency, feature correlation in training data, capacity difference of hardware components, and many other factors. 
  
- Discussed three models of consistency: sequential, eventual, bounded delay. 

- Parameter server is able to cover orders of magnitude more data on orders of magnitude more processors than any other published system. It simplifies the development of distributed machine learning applications. It enabled LDA (Latent Dirichlet Allocation) models with 10s of billions of parameters to be inferred from billions of documents, using up to thousands of machines

- Natural but also clear division of BOTH: the input data and the parameters (in the form of key, value pairs) between many worker nodes and parameter server nodes, respectively. 
  - This approach allows them to scale the computing resources, the data used for training machine learning models and the models themselves (with billions of parameters).

- Both key/parameter caching and data compressing system-level optimization are generalized to user-defined filters.

## Weak Points:

- Do not offer higher-level general-purpose building blocks such as model partitioning strategies, scheduling, managed communication that are key to simplifying the adoption of a wide range of ML methods. 

- They used the traditional techniques from distributed systems with some small improvements (vectors clocks compressed taking into account that not many vector clocks diverge), snappy compression applied on messages - to compress zeros as user-defined filters may zero out large fraction of the parameters.
