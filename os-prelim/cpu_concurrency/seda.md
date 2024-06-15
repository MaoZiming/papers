# SEDA: An Architecture for Well-Conditioned, Scalable Internet Services
This paper presents staged event-driven architecture (SEDA), a new design for highly concurrent Internet services. SEDA is intended to support massive concurrency demands and simplify constructions of well-conditioned services. The key innovations include a combination of thread-based and event-based model, and use resource allocators to adapt resources at each stage according to the workloads. 

Key components include 
1. **Stages**: The application is divided into multiple stages, each associated with an incoming event queue.
2. **Dynamic Resource Controllers**: Controllers dynamically adjust the number of threads and the batching sizes within each stage to adapt to workload changes.
3. Explicit Queues: Interconnect different stages to buffer and manage events.
4. Asynchronous I/O Primitives: Non-blocking I/O operations to keep the system responsive.

## Limitations 
Cons of overall event-based system include the difficulty to understand cause-effect relationship, and stack ripping (i.e. manually save and restore live state is a pain). The trade-offs were mentioned in the Cappricio paper. 
