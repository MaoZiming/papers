# Capriccio: Scalable Threads for Internet Services (2003) 
This paper proposes Capriccio, a scalable thread package for use with high-concurrency servers. The main proposal is to use **user-level threads** instead of event-based models (i.e. SEDA) to achieve high performance without sacrificing the ease of programming. 

Its approach is three-fold: 

1. **Scalability**: Capriccio uses user-level threads coupled with cooperative scheduling to manage many threads efficiently. User-level threads are preferred over kernel threads for their flexibility and performance advantages, like reduced synchronization overhead.

2. **Linked Stack**: Traditional stack allocation methods are not suitable for programs managing many threads. Capriccio uses a dynamic stack allocation technique that employs linked chunks, which are managed through compile-time analysis and runtime checks.

3. **Resource-aware Scheduler**: The scheduler views the application as a sequence of stages, separated by blocking points, forming a "blocking graph". This graph is used to make application-specific scheduling decisions to maximize resource utilization without reaching bottlenecks.

## Limitations 
* Does not completely eliminate kernel crossings.
* Performance not necessarily better than event-based systems.

## Connections 
Scheduler Activations: Capriccio’s use of user-level threads echoes the sentiment in this paper about effective kernel support for user-level management of parallelism.

SEDA: Both papers tackle the problem of internet service scalability but approach it differently - Capriccio uses user-level threads while SEDA employs an event-based architecture.

Scheduling: Proportional Share - The resource-aware scheduler in Capriccio can be related to proportional share scheduling concepts, especially in how it seeks to allocate resources effectively.
