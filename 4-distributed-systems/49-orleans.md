# Orleans

Link: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/Orleans-MSR-TR-2014-41.pdf

June 30th, 2024.

- High-scale interactive services demand high throughput with low latency and high availability, difficult goals to meet with the traditional stateless 3-tier architecture. The actor model makes it natural to build a stateful middle tier and achieve the required performance. However, the popular actor model platforms still pass many distributed systems problems to the developers.
- The Orleans programming model introduces the novel abstraction of virtual actors that solves a number of the complex distributed systems problems, such as reliability and distributed resource management, liberating the developers from dealing with those concerns. At the same time, the Orleans runtime enables applications to attain high performance, reliability and scalability.
- This paper presents the design principles behind Orleans and demonstrates how Orleans achieves a simple programming model that meets these goals. We describe how Orleans simplified the development of several scalable production applications on Windows Azure, and report on the performance of those production systems.

## Key Points

- First, an Orleans actor always exists, virtually. It cannot be explicitly created or destroyed. Its existence transcends the lifetime of any of its in-memory instantiations, and thus transcends the lifetime of any particular server.
- Orleans actors are automatically instantiated: if there is no in-memory instance of an actor, **a message sent to the actor causes a new instance to be created on an available server**. An unused actor instance is automatically reclaimed as part of runtime resource management. 
- Third, the location of the actor instance is transparent to the application code, which greatly simplifies programming.
- Fourth, Orleans can automatically create multiple instances of the same stateless actor, seamlessly scaling out hot actors.