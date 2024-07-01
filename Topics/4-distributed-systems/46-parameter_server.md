# Parameter Server

This paper is about one of the two main components of machine learning, namely optimization (the architecture and model selection being the other major component).

The asynchronous parallel optimization recently received many successes and broad attention in machine learning and optimization. It is mainly due to that the asynchronous parallelism largely reduces the system overhead comparing to the synchronous parallelism. The key idea of the asynchronous parallelism is to allow all workers work independently and have no need of synchronization or coordination. The asynchronous parallelism has been successfully applied to speedup many state-of-the-art optimization algorithms including stochastic gradient descent

## Strong Points

- A core goal of the parameter server framework is to capture the benefits of GraphLab’s asynchrony (to schedule communication using a graph abstraction) without its structural limitation that impede scalability and elasticity (namely, the lack of global variable and state synchronization/sharing as an efficient first-class primitive).

- Machine Learning specialized optimizations: message compression, replication, and variable consistency models expressed via dependency graphs. They achieve synergy by picking the right systems techniques, adapting them to the machine learning algorithms, and modifying the machine learning algorithms to be more systems friendly. The trade-off between the system efficiency and algorithm convergence rate depends on: algorithm’s sensitivity to data inconsistency, feature correlation in training data, capacity difference of hardware components, and many other factors. Algorithm designer can define consistency models via the “bounded delay” method. Moreover, the scheduler may increase of decrease the maximal delay according to the runtime progress to balance system efficiency and convergence of the underlying optimization algorithm.

- Parameter server is able to cover orders of magnitude more data on orders of magnitude more processors than any other published system. It simplifies the development of distributed machine learning applications. It enabled LDA (Latent Dirichlet Allocation) models with 10s of billions of parameters to be inferred from billions of documents, using up to thousands of machines

- Natural but also clear division of BOTH: the input data and the parameters (in the form of key, value pairs) between many worker nodes and parameter server nodes, respectively. This approach allows them to scale the computing resources, the data used for training machine learning models and the models themselves (with billions of parameters). It can also work by communicating only a range of keys/parameters at a time.

- Both key/parameter caching and data compressing system-level optimization are generalized to user-defined filters.

## Weak Points:

- Do not offer higher-level general-purpose building blocks such as model partitioning strategies, scheduling, managed communication that are key to simplifying the adoption of a wide range of ML methods. In general, systems supporting distributed ML manifest a unique trade-off between efficiency, correctness, programmability, and generality.

- They used the traditional techniques from distributed systems with some small improvements (vectors clocks compressed taking into account that not many vector clocks diverge), snappy compression applied on messages - to compress zeros as user-defined filters may zero out large fraction of the parameters.

- Many trade-offs: accuracy of the machine learning model and the model for consistency. They don’t provide any graphs on how their methods hurt the machine learning models accuracy!

- Bounded delays were discussed in many previous paper (also by Alexander J. Smola). There is a very similar paper to this one (repeating even the figures) in the NIPS conference.
