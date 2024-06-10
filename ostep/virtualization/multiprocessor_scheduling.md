# Multiprocessor Scheduling

- Multiprocessor systems are increasingly commonplace 
- Multiple CPU cores are packed onto a single chip, 
- The difference between a single-CPU hardware and multi-CPU hardware: centers around the use of hardware caches, and exactly how data is shared across multiple processors 
- Cache: a notion of locality 
  - Temporal locality: when data is accessed, it is likely to be accessed again in near future 
  - Spacial locality: if the program access a data item at address x, it is likely to access data items near x as well 

## Problem #1: Cache Coherence 
- Multi-processor problem: cache coherence!  
- Basic solution provided by the hardware (i.e. hardware protocol) 
- Bus snooping: each cache pays attention to memory updates by observing the bus that connects them to main memory 
  - When CPU see update for data item it holds in cache, notice the change
  - Then, either **invalidate** its copy (remove from cache) or **update** it (put new value in cache)
  - Write-back cache makes this more complicated!

## Problem #2: Synchronization
- OS still need to worry about stuff when they access shared data: concurrency! 
  - Cross-CPU access shared data items or data structures 
  - Use mutual exclusion primitives (such as locks) to guarantee correctness 
    - But as # of CPUs grow, access to a synchronized shared data structure becomes quite slow 
- Other approaches such as building lock-free data structures, are complex and only used on occasion; see the chapter on deadlock in the piece on concurrency for details

## Problem #3: Cache Affinity 
- Notion: A process running on a particular CPU builds up a fair bit of state in caches (and TLBs) of the CPU 
  - The next time the process runs, it is often advantageous to run this process on the same CPU again 
  - As it will run faster if some of its state already present in caches on that CPU 
  - Otherwise has to reload the state 

## Building a scheduler for multiprocessor system 
- 3.1 Single-Queue Multiprocessor Scheduling (**SQMS**) 
  - Putting all jobs needed to be scheduled into a single queue 
  - pros: simplicity
    - Does not require much work to take an existing policy that picks the best job to run next and adapt it to work on more than one CPU
  - Cons
    - **Scalability**
      - Need locking to ensure that when SQMS code access the single queue, to proper outcome arises
    - **Performance**
      - Lock reduces performance as # of CPUs grows
      - I.e. contention for a single lock increases, the system spends more time in lock overhead
    - **Cache affinity**
      - Each CPU simply picks the next job to run from globally shared queued, each job ended up bouncing around from CPU to CPU
  - most SQMS schedulers include some kind of affinity mechanism to try to make it more likely that process will continue to run on the same CPU if possible.
- 3.2 Multi-Queue Scheduling (MQMS) 
  - Notion: multiple scheduling queues, one per CPU, scheduled independently
  - When a job enters the system, it is placed on exactly one scheduling queue, according to some heuristics (e.g., random, picking one with fewer jobs) 
  - Avoid the problems of information sharing and synchronization in the single-queue approach 
  - Pros
    - Inherently more **scalable**
        - As # of CPU grows, so does the number of queues
        - Lock and cache contention should not be a central problem
    - Intrinsically provide **cache affinity**
        - Jobs stay on the same CPU, reap the advantage of reusing cached contents again
  - Cons
    - Load imbalance 
    - Dealing with load imbalance: **migrate** a job from one CPU to another
    - Continuous migration of one or more jobs.
      - Many possible strategies for migration.
    - Basic approach: **work stealing**
        - A (source) queue that is low on jobs will occasionally peek at another (target) queue, to see how full it is
        - If target queue is notably more full, then the source steal one or more jobs from the target to help balance load
        - If you look around at other queues too often,you will suffer from high overhead and have trouble scaling, which was the entire purpose of implementing the multiple queue scheduling in the first place!
        - Compared to job migraiton:
          - Decentralized approach;
          - Pull mechanism (local queue "pull")
- #3: Linux Multiprocessor Schedulers 
  - Priority-based scheduler (similar to MLFQ)
    - Changing a process’s priority over time and schedule those with highest priority
    - Interactivity is the primary focus
  - Multi-queue 
  - O(1) scheduler
    - multi-queue
    - Similar to MLFQ and priority-based
  - Completely Fair Scheduler (CFS)
    - Deterministic proportional-share approach (similar to Stride scheduling) 
    - Multi-queue 
    - like stride scheduling. 
  - BF Scheduler (BFS)
    - Single queue 
    - Proportional-share, with Earliest Eligible Virtual Deadline First (EEVDF) [SA96].