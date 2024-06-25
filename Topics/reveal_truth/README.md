# System Design Principals
* **Computer System Design (1983)**
  *   Good system: functionality, speed, fault tolerance
  *   Functionality: keep it simple, get it right, keep basic interface stable, specialize first (don't generalize) 
  *   Speed: split resource instead of sharing, cache, batching
  *   FT: make actions atomic, log 
* **End-to-end principal (1984)** 
  *   Applications know the best what they need
  *   Underlying systems should provide only a minimal set of functionalities that are absolutely necessary
  *   Application-specific features are kept at communication endpoint 
  *   E.x. Exokernels, error handling in file transfer, end-to-end encryption
* **Software Engineering Advice (2007)**
  *   Identify the general problem, address them in general way
  *   Understand data access
      *  Disk (seeks, sequential reads), memory (cache, branch predictor, etc.)
      *  RPCs: how much data sent / received, whether saturate network interface and rack switch 
  *   Design for low latency
      *  CPU fast, memory / bandwidth precious: encode data (e.x. compression)
      *  Use parallelism (i.e. multi-thread) 
      *  Latency tail: worry about variance!
      *  Use higher priorities for interactive requests     
  *   Most: eventual consistency
  *   FT: fail over to replicas, detection, online profiling 
* **The Tail at Scale (2013)**
  *   Insights
      *  Rare performance hiccups affect many requests in large-scale DS 
      *  Eliminate all sources of latency variability is impractical in shared env 
         *   shared resources (e.x. CPU, memory, network; network switches, shared FS)
         *   maintenance activities (e.x. log compaction, garbage collection)
         *   daemons, queueing, power limits, energy management (e.x. in devices)
      *  Component-level variability amplified by scale
  *  Use "tail-tolerant" technique
      *  Idea: form a predictable whole out of less-predictable part
      *  Reduce component variability
          *  Service classes: interactive first
          *  Reduce head-of-line blocking: time slicing
          *  Manage background tasks: throttle, trigger it when low load, use smaller ops
      *  Tail-tolerant techniques
          *  Within-Request Short-Term 
              *  (1) Hedged request
                 *   first send to one "most appropriate" replica, then fall back to secondary after brief delay
              *  (2) Tied request
                 *   deal with queueing delays 
                 *   enqueue req copies in multi-server, servers comm updates on copy status
          *  Cross-Request Long-Term
              *  (1) Micro-partitions: more than # of machines, assignment, load balancing
              *  (2) Selective replication: predict "hot"
              *  (3) Latency-induced probation
                  *  put slow machine on probation
                  *  issue shadow request (allow rejoin when problem abates)
          *   Information retrival: return "good enough" result, canary request 
