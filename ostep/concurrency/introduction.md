# Concurrency: An Introduction 

### State of a single thread

- A program counter (PC)
    - Tracks where the program is fetching instructions from
- A private set of registers
    - Uses for computation
    - Thus, when two threads are running on a single processor, when switching from T1 to T2, context switch must take place
    - I.e. register state of T1 must be saved and register state of T2 restored
    - States: **thread control blocks (TCBs)**
        - V.s. process control block (PCB)
- A stack per thread
    - Thread-local storage, i.e. the stack of the relevant thread
- This model is usually okay since the stacks do not generally have to be very large (with exceptions on heavy use of recursion)
- Why not multi-process?
    - Sharing the same address space makes it easier to share data 
    - Processes are more for logically separate tasks where little sharing of data structures in memory is needed
- Multiple threads executing this code can result in race condition, this code is critical section 
- What we want for this code is mutual exclusion 
