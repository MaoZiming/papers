## The Tail at Scale (2013)

Link: https://cacm.acm.org/research/the-tail-at-scale/

Read: July 5th, 2024. 

  *   Insights
      *  Rare performance hiccups affect many requests in large-scale DS 
      *  Eliminate all sources of latency variability is impractical in shared env 
         *   shared resources (e.x. CPU, memory, network; network switches, shared FS)
         *   maintenance activities (e.x. log compaction, garbage collection)
         *   daemons, queueing, power limits, energy management (e.x. in devices)
      *  Component-level variability amplified by scale
      *  It is sometimes useful for the system to break long-running requests into a sequence of smaller requests to allow interleaving of the execution of other short-running requests; 
  *  Use "tail-tolerant" technique
      *  Idea: form a predictable whole out of less-predictable part
      *  Reduce component variability
          *  Service classes: interactive first
          *  Reduce head-of-line blocking: time slicing
          *  Manage background tasks: throttle, trigger it when low load, use smaller ops
      *  Tail-tolerant techniques
          *  Within-Request Short-Term 
              *  (1) Hedged request
                 *  A simple way to curb latency variability is to issue the same request to multiple replicas and use the results from whichever replica responds first. 
                 *   first send to one "most appropriate" replica, then fall back to secondary after brief delay
                 *   The client cancels remaining outstanding re- quests once the first result is received.
              *  (2) Tied request
                 *  "Mitzenmacher10 said allowing a client to choose between two servers based on queue lengths at enqueue time exponentially improves load-balancing performance over a uni- form random scheme."
                 *   deal with queueing delays 
                 *   enqueue req copies in multi-server, servers comm updates on copy status
                 *    We call requests where servers perform cross-server sta- tus updates “tied requests.” 
                 *    The simplest form of a tied request has the cli- ent send the request to two different servers, each tagged with the identity of the other server (“tied”). When a request begins execution, it sends a cancellation message to its counterpart. The corre- sponding request, if still enqueued in the other server, can be aborted imme- diately or deprioritized substantially.
          *  Cross-Request Long-Term
              *  (1) Micro-partitions: more than # of machines, assignment, load balancing
                 *  Do dynamic assignment and load balancing of these partitions to particular machines. Load balancing is then a matter of moving responsibility for one of these small partitions from one machine to another.
              *  (2) Selective replication: predict "hot"
              *  (3) Latency-induced probation
                    *  then do dynamic assignment and load balanc- ing of these partitions to particular ma- chines. Load balancing is then a matter of moving responsibility for one of these small partitions from one machine to another.
                  *  put slow machine on probation
                  *  issue shadow request (allow rejoin when problem abates); collecting statistics on their latency so that they can be reincorperated into the service.
          *   Information retrival: return "good enough" result, canary request 
          *   Carnary requests:
              *   rather than initially send a request to thousands of leaf servers, a root server sends it first to one or two leaf servers. The remaining servers are only queried if the root gets a successful response from the canary in a reasonable period of time. If the server crashes or hangs while the canary request is outstanding, the system flags the request as poten- tially dangerous and prevents further execution by not sending it to the remaining leaf servers.


> A common technique for reducing la- tency in large-scale online services is to parallelize sub-operations across many different machines, where each sub-op- eration is co-located with its portion of a large dataset. Parallelization happens by fanning out a request from a root to a large number of leaf servers and merg- ing responses via a request-distribution tree. These sub-operations must all complete within a strict deadline for the service to feel responsive.

> Missing in this discussion so far is any reference to caching. While effec- tive caching layers can be useful, even a necessity in some systems, they do not directly address tail latency, aside from configurations where it is guaranteed that the entire working set of an applica- tion can reside in a cache.