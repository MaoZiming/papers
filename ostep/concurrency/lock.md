# Lock

- POSIX uses `mutex` as the name for lock
    - Stands for mutual exclusion between threads
    - If one thread is in the CR, excludes others from entering until it has completed the section

## Controlling Interrupts

- Earliest solution for single-processor system

```c
void lock() {
    disable_interrupts();
}
void unlock() {
    enable_interrupts();
}
```

- Pros: simplicity
- Cons
    - Requires us to allow any calling thread to perform privileged operation (turning interrupts on and off), and trust that this facility is but abused
        - Requires too much trust in applications
    - Does not work in multiprocessors
        - Multi-threads running on different CPU, and entering the critical section, it does not matter whether interrupts are disabled
    - Interrupts can be lost
        - I.e. CPU missed the fact that a disk device has finished a read request
    - Inefficient
        - Code that masks or unmasks interrupts tends to be executed slowly
  
- Pros: simplicity
- Cons
    - Allow calling thread to perform privileged instructions and trust that it is not abused
    - On multi-processor, each threads running on different CPUs, and each try to enter the same critical sections, does not matter whether interrupts are disabled
## A Failed Attempt: Using Loads / Stores

- Problems:
  - Correctness
    - Both threads are able to set the flag to 1 and able to enter the critical section
  - Performance 
    - Spin-waiting: endlessly check the value of the flag 

## Hardware support for locking: Test-And-Set (Atomic Exchange) 

- Returns the old value pointed to by the old_ptr, simultaneously update said value to new
- This sequence of operation is done atomically
    - By making bot the test (of the old lock value)
    - And set (of the new value)
    - A single atomic operation, we ensure that only one thread acquires the lock
- Spin lock
    - Spins, using CPU cycles, until the lock becomes available
    - On a single processor: requires a **preemptive scheduler** (i.e. one that will interrupt a thread via a timer, in order to run different thread)
    - Evaluating spin lock
        - Correctness: mutual exclusion?
            - YES!
        - Fairness: how fair is a spin lock to a waiting thread?
            - NO FAIRNESS GUARANTEES
            - Might be starvation
        - Performance: what are the costs of using a spin lock?
            - Single CPU: performance is painful
                - Thread holding the lock is preempted within a critical section
                - Scheduler might run every other thread trying to acquire the lock
                - Each threads spin for duration of the time slice before giving up the CPU
            - Multi-CPU: works reasonably well if # of threads ~ # of CPUs

## Compare-And-Swap (CAS)
- Test whether the value at the address specified by `ptr` is equal to `expected`
- If so, update the memory location pointed to by `ptr` with the new value
- If not, do nothing
- In either case, return original value at that memory location, allow the code calling this call knows it succeeds or not
- Pros
  - More powerful instruction than test-and-set

## Load-Linked and Store-Conditional 
- Load-linked: fetch a value from memory and place it in register
- Store-conditional: succeed if no intervening store to the address has taken place
    - Success: returns 1 and update value at `ptr` to `value`
    - Fail: `ptr` is not updated and 0 is returned

## Fetch-and-Add 
- Atomically increments a value while returning the old value at a particular address
    - Can use this to build a ticket lock
- Difference
    - Ensures progress for all threads
    - Once a thread is assigned with a ticket value, will be scheduled in some point in the future

## Too much spinning: what now? 

- Problem gets worse with more threads contending for a lock
- N - 1 time slice may be wasted in similar manner, spinning and waiting for a single thread to release the lock
- Problem: how can we develop a lock that doesn’t needlessly waste time spinning on the CPU?
    - **Hardware support alone cannot solve the problem**
    - **Need OS support too!**

## Simple Approach: Just Yield
- `yield()`: a sys call that moves the caller from running state to ready state, and promote other thread to run
    - De-schedules itself
    - However approach still very costly: run-and-yield for every other thread, and cost of context switch can be substantial

## Using Queues: Sleeping Instead of Spinning 
- OS support and a queue to keep track of which threads are waiting to acquire the lock 
  - Queues to control who gets the lock next and thus avoid starvation 
- `park()`: put a calling thread to sleep
  - `park()` is used to block the current thread. It puts the thread into a waiting state until it is explicitly unblocked.
  - The thread remains parked until it is unparked by another thread or it is interrupted.
  - This method does not release any locks held by the thread when it is called.
- `unpark()`: wake a particular thread
    - `unpark` is used to unblock a thread that was previously parked. It makes the specified thread eligible for scheduling.
    - If the specified thread is not currently parked, a future park call will return immediately.


## Others
- Futex from Linux
- Two phase lock
    - First phase: spin for a while, hope to acquire the lock
    - Second phase: caller put to sleep, only woken up when the lock becomes free
  
## Crux: How to add locks to data structures

- When given a particular data structure, how should we add locks to it, in order to make it work correctly?
- How do we add locks such that data structure yields high performance, enabling many threads to access the structure at once, i.e. concurrently?