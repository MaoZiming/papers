# Synchronization 
- Implementation of threads and locks (disabling interrupts and atomic operations like CompareAndSwap)
- How to use semaphores, locks, and condition variables

## Threads & Locks

- **Thread:** new abstraction for single running process
    - A multi-thread program has more than one point of execution
        - I.e. multiple PCs, each of which is being fetched and executed from
    - A separate process, except once difference
        - They share the same address space and thus can access the same data
    - *State* of a single thread
        - Program counter (PC)
        - Private set of registers: used for computation
            - States are stored in thread control blocks (TCBs)
        - A stack per thread: thread-local storage (local variables, arguments, return values, etc.)
- **Why threads?**
    - Parallelism: multi-CPUs
    - To avoid blocking program due to slow I/O
        - If block, switch to other thread
        - Enable overlap of I/O with some other activities
    - Why not multi-process?
        - Sharing the same address space, easier to share data
- **Why it get worse?** Shared data and synchronization
    - Multiple threads executing this code can result in race condition, this code is **critical section**
    - What we want for this code is **mutual exclusion** and “**atomicity**”
- How to build synchronization?
    - Use hardware support
    - With OS help
- **How to build a lock?**
    - We would like to execute a series of instructions atomically, but due to the presence of interrupts on a single processor (or multi-thread on multi-processor), we couldn’t
    - We deal with this problem directly with the notion of lock
        - Programmers annotate source code with locks
        - Putting them around critical sections
        - And ensure that critical section executes as if it were a single atomic instruction
    - **#1: disabling interrupt for critical sections**
        - Steps
            - Code inside the critical section will not be interrupted, and will execute as if it were atomic
            - When we finish, reenable interrupts (via hardware instructions)
        - Pros: simplicity
        - Cons
            - Allow calling thread to perform privileged instructions and trust that it is not abused
            - On multi-processor, each threads running on different CPUs, and each try to enter the same critical sections, does not matter whether interrupts are disabled
    - **#2: Compare and Swap [hardware support]**
        - Basic Idea
            - Test whether the value at the address specified by `ptr` is equal to `expected`
            - If so, update the memory location pointed to by `ptr` with the new value
            - If not, do nothing
            - In either case, return original value at that memory location, allow the code calling this call knows it succeeds or not
        
        <img width="584" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/ea5d6eb1-0dd9-4faa-bdd1-53543284876f">

        
        - Pros: more powerful instruction than test-and-set (i.e. in lock-free synchronization) setup

## Locks
- POSIX call lock: mutex
    - To provide the mutual exclusion
- Example of lock as below
<img width="624" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/ea5b04af-772c-4031-9b99-656bc2fb24c1">

## Condition Variables
- **Condition variable**
    - Def: explicit queue that threads can put themselves on when some state of execution (i.e. some **condition**) is not as desired (by **waiting** on the condition)
    - Some other thread, when it changes said state, can then wake one (or more) of those waiting threads and allow them to continue (by **signaling** on condition)

<img width="621" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/14618f97-20cc-430d-8a8a-954492efc29c">


## Semaphore
- A semaphore: an object with an integer value that we can manipulate with two routines
    - `sem_wait()`
    - `sem_post()`

```c
// initialize a semaphore 
#include <semaphore.h> sem_t s;
sem_init(&s, 0, 1); 

// definiton of wait and post 
int sem_wait(sem_t *s) {
	// decrement the value of semaphore s 
	// by one wait if value of semaphore s is negative
} 

int sem_post(sem_t *s) { 
	// increment the value of semaphore s by one 
	// if there are one or more threads waiting, wake one 
} 

// A binrary semaphore (i.e. a lock)
sem_t m;
sem_init(&m, 0, X); // initialize to X; what should X be?
sem_wait(&m);
// critical section here
sem_post(&m);
```

- Binary semaphore
<img width="611" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/d88ad80c-2813-4472-83a9-1944e83e4253">

