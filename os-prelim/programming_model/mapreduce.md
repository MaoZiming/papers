# MapReduce 
Map Reduce is a simple ata-parallel programming model designed for scalability and fault-tolerance. It provides a limited but general functional API including map, reduce, and sort, and it requires no other synchronization and communication. It achieve fault recovery, and straggler mitigation through retries. 

## Key Motivation 
- Background: MPI (message passing interface)
    - Expressive programming model, more general (v.s. MR: limited)
    - Not inherently fault tolerance: if a node fails, can cause entire app crash
    - Fine-grained synchronization and communication
- Motivation: build google web search (i.e. crawl documents, build inverted indexes, etc.), need for
    - Automatic parallelization
    - Network, disk optimization
    - Handling of machine failures

### Assumptions & Design Goals

1. Failures are norm
2. Local storage is cheap
3. Commodity networking, less bisection bandwidth 
4. Replicated FS 

## Key Techniques
- Data type: each record is (key, value)
- Map function: $(K_{in}, V_{in}) ->list(K_{inter}, V_{inter})$
    - Take one input KV pair and output a set of intermediate KV pairs
- Shuffle phase
    - Data exchange step between Map and Reduce tasks
    - After Map tasks finish processing, intermediate results are partitioned, grouped by key, and transferred to Reducers
    - Potential problems
        - Data skew: one key with large # of values, overload single Reducer
        - Network bandwidth: too much intermediate data transferred over the network
        - Disk I/O: might be the bottleneck
    - Not always needed
        - I.e. no reduce phase, the application doesn’t require grouping by key, then shuffle might be skipped
        - Applications without shuffle: grep, or simple transformation on the data, data sampling
- Reduce function: $(K_{inter}, list(V_{inter}))->list(V_{out})$
    - Accept one key and all the values for that key and merges the values together
 
## Fault Tolerance 
- **If a task crashes**
    - Retry on another node (note: inputs can still be read bc of replication)
    - If the same task repeatedly fails, end the job
- **If a node crashes**
    - Detect failure via periodic heartbeats
    - Relaunch its current tasks on other nodes
    - What about task inputs? File system replication
- **If master failures**
    - Checkpoints and retry the job
    - Probability of master failing is low
- If a task is going slowly
    - Straggler: a machine takes an unusually long time to complete one of the last few map or reduce task in the computation
    - Launch second copy of task on another node
    - Take the output of whoever finishes first

## Limitations 
- Not every program can be expressed via this model (i.e. iterative, low-latency chained events like PageRank, conjugate gradient, etc.)
- Failures of master aborts the computation, up to the client app to issue the retry
