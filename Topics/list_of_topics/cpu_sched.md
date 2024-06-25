# CPU Scheduling 
- Classic policies: FCFS (First-Come, First-Served), Round Robin, Priorities, SRTF (Shortest Remaining Time First), MLFQ (Multi-Level Feedback Queue), and EDF (Earliest Deadline First)
- Starvation (including priority donation/inheritance) and deadlock (including prevention techniques)
- Look at how multi-core scheduling works as well
    - [Multiprocessor Scheduling (Advanced)](https://www.notion.so/Multiprocessor-Scheduling-Advanced-e41c3d1425e24b5189b9b1866fa14b46?pvs=21)

## Metrics

1. **Turnaround time:** the time at which the job completes - the time which the job arrived
2. **Response time:** the time from when the job arrives in the system to the first time it is scheduled 

## Classic Policies

| Policies  | Explain  |
| --- | --- |
| FCFS (FIFO)  | First in, First Out |
| SJF | Shortest Job First | 
| STCF | Shortest Time-to-Completion First | 
| RR | Round Robin | 
| MLFQ | Multi-level Feedback Queue | 
| EDF | Earliest Deadline First | 
| Proportional Share Scheduler | Fair Share | 

### FCFS (FIFO) 
First in, First Out.
Cons:
- Average turnaround time can be high! 
- Convey effect: a number of relatively-short potential consumers of resource get queued behind a heavyweight resource consumer

### SJF 
Improvement on FIFO: run the shortest first job first. 

* Pros: optimal scheduling algorithm for turnaround time given assumptions that jobs arriving at the same time. 
* Cons: same convey problem if we relaxed this assumption (i.e. schedule long A, then short B,C arrive).

### STCF 
Add preemption to SJF. Any time a new job enters the system, the STCF scheduler determines which of the remaining jobs has the least time left, and schedules that one. 

### RR 
Instead of running jobs to completion, RR runs a job for a time slice (i.e. scheduling quantum), then switches to next job in the run queue. Note: length of time slice is critical (i.e. shorter, better response time, but higher context switch overhead).
* Pros: built for high response time, fair
* Cons: perform bad for turnaround time, even worse than FIFO in many cases  

### MLFQ 
Problem MLFQ tries to solve is that we want to optimize turnaround and minimize response. we don’t know anything about a process (i.e. job length, etc.). The basic ideas: 
1. Number of distinct queues with different priority level: higher priority given more CPU time, lower given less
2. Priority adjusted dynamically based on its behavior, use a feedback mechanism
3. Each queue is assigned a time slice determine how much CPU time a process in that queue is allowed to use before pre-empted and moved to a lower level queue

* Pros: more flexible, allow different process to move between priorities, prevent starvation by increasing process’s priority
* Cons: many tunable knobs: # of queues, time slice per queue, how often should priority be boosted, etc.

### EDF 
Assign priorities to the task according to the absolute deadline. The task whose deadline is closet gets the highest priority. 
* Pros: efficient in real time system, dynamic
* Cons: priority inversion (i.e. a low-priority task can block a higher priority task from being executed, if blocking?) 

### Proportional Share (Fair-Share) Scheduler  
Instead of optimizing for turnaround or response time, scheduler might instead try to guarantee that each job obtain a certain percentage of CPU time.

1. _Lottery Ticket_: use tickets to represent the share of a resource that a process should receive 
- Lightweight, randomness 
- But occasionally not deliver right proportions 

2. _Stride Scheduling_: not use randomness 
- Stride: inverse in proportion to # of tickets it has 
- use the stride and pass to determine which process to run next; pick process to run that has the lowest pass value so far, when you run a process, increment its pass counter by stride 

3. _CFS_ 
- Linux CFS: highly efficient and scalable fair-share scheduler 
- Goal: fairly divide CPU evenly among competing processes 
- use virtual runtime (vruntime), weight with niceness to assign priorities 
- weighted round-robin with dynamic time slices
  
## Starvation

- Starvation occurs when a process is perpetually denied necessary resources.
- Problem: in MLFQ
    - If there are “too many” interactive jobs in the system, they will combine to consume all CPU time, thus long running jobs will never receive any CPU time (they starve)
- **Solution**
    - MLFQ: priority boost
        - Problem: Low-priority tasks are ignored
        - The simple idea here is to periodically boost the priority of all the jobs
        in system.
        - **Rule 5:** After some time period S, move all the jobs in the system to the topmost queue.
    - **Priority donation / inheritance**
        - Problem: priority inversion
            - Higher-priority task blocked by lower priority task that holds a needed resource (i.e. lock)
        - Low priority task inherits the high priority of a task it is blocking
        - Here's a simple example:
            1. **Low-Priority Task (L)** owns a mutex.
            2. **High-Priority Task (H)** tries to acquire the same mutex but is blocked because it's owned by **L**.
            3. **Medium-Priority Task (M)** preempts **L** because **M** has a higher priority than **L** but lower than **H**.
            
            Now, **H** is waiting for **L** to release the mutex, but **L** itself is waiting for CPU time because it's preempted by **M**. This is priority inversion.
            
            In a system with Priority Donation/Inheritance:
            
            1. When **H** is blocked by **L**, the system temporarily elevates the priority of **L** to that of **H**.
            2. Now, **L** can preempt **M** because it has "inherited" the higher priority.
            3. **L** quickly finishes its work, releases the mutex.
            4. **H** can now acquire the mutex and proceed, and **L** returns to its original low priority.

## Deadlock
- Deadlock is a specific condition where two or more processes are unable to proceed because each is waiting for the other to release a resource.
- Prevention
    - Resource allocation graph: use directed graph to represent resource allocation, avoid cycles
    - Timeout: set maximum time for acquiring resources
