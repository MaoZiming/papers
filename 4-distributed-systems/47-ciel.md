# Ciel: A Universal Execution Engine for Distributed Data-Flow Computing

Link: https://www.usenix.org/legacy/event/nsdi11/tech/full_papers/Murray.pdf

Read: June 30th, 2024. 

- Ciel introduces **dynamic** task graphs which are extended at runtime, as opposed to static graphs in earlier models (e.x. MapReduce, Dryad). 
- It allows for more adaptive and flexible task execution. This dynamic approach, coupled with a runtime that can adjust execution strategies based on current conditions, makes it possible to support a wider range of applications efficiently. 
- Examples of usecases include iterative, low-latency chained events like k-means, E-M, PageRank, and so on. 

- SkyWriting: a scripting language that allows the straightforward expression of iterative and recursive task-parallel algorithms using imperative and functional language syntax.
  - However CIEL extends previous models by **dynamically building the DAG** as tasks execute. As we will show, this conceptually simple extension— allowing tasks to create further tasks—enables CIEL to support data-dependent iterative or recursive algorithms
- In MapReduce, the data flow is limited to a bipartite graph parameterised by the number of map and reduce tasks; Dryad allows data flow to follow a more general directed acyclic graph (DAG), but it must be fully specified before starting the job. 
- In general, to support iterative or recursive algorithms within a single job, we need data-dependent control flow—i.e. the ability to create more work dynamically, based on the results of previous computations. 
- At the same time, we wish to retain the existing benefits of task-level parallelism: transparent fault tolerance, locality-based scheduling and transparent scaling.
- Having **data-dependent control flow** means that each computation doesn't need to be pre-determined. 
- **Primitives**:
  - **Objects**: Goal of a CIEL job is to produce one or more output objects. An object is an unstructured, finite-length sequence of bytes.
  - **References**: A reference comprises a name and a set of locations (e.g. host name-port pairs) where the object with that name is stored. The set of locations may be empty: in that case, the reference is a future reference to an object that has not yet been produced. Otherwise, it is a **concrete** reference, which may be consumed.
  - **Tasks**:  A CIEL job makes progress by executing tasks. A task is a non-blocking atomic computation that executes completely **on a single machine**. The task becomes runnable when all of its dependencies become **concrete**. 
    - A task can publish one or more objects, by creating a concrete reference for those objects. In particular, the task can publish objects for its expected outputs, which may cause other tasks to become runnable if they depend on those outputs.
    - To support data-dependent control flow, however, a task may also **spawn new tasks** that perform additional computation.
  - ![alt text](images/47-ciel/ciel-cluster.png)
    - Since task inputs and outputs may be very large (on the order of gigabytes per task), all bulk data is stored on the workers themselves.

## SkyWriting
  - Skywriting functions are pure: functions cannot have side-effects, and all arguments are passed by value. 
  -  The dereference (unary- ) operator can be applied * to any reference; it loads the referenced data into the Skywriting execution context, and evaluates to the resulting data structure.
  -  A Skywriting task has a single output, which is the value of the expression in the return statement. On termination, the runtime stores the output in the local object store, publishes a concrete reference to the object, and sends a list of spawned tasks to the master, in order of creation.
  -  `spwawn`, `execute` `spawn_exec` . The results of `spawn` and `spawn_exec` are first-class futures. A Skywriting task can pass the references in its return value or in a subsequent call to the task-creation functions.