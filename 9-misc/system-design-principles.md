# System Design Principles
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

