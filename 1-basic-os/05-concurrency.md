# Synchronization

## Threads & Locks

* **Thread:** new abstraction for single running process
  * A multi-thread program has more than one point of execution
    * I.e. multiple PCs, each of which is being fetched and executed from
  * A separate process, except once difference
    * They share the same address space and thus can access the same data
  * *State* of a single thread
    * Program counter (PC)
    * Private set of registers: used for computation
      * States are stored in thread control blocks (TCBs)
    * A stack per thread: thread-local storage (local variables, arguments, return values, etc.)
* **Why threads?**
  * Parallelism: multi-CPUs
  * To avoid blocking program due to slow I/O
    * If block, switch to other thread
    * Enable overlap of I/O with some other activities
  * Why not multi-process?
    * Sharing the same address space, easier to share data
* **Why it get worse?** Shared data and synchronization
  * Multiple threads executing this code can result in race condition, this code is **critical section**
  * What we want for this code is **mutual exclusion** and “**atomicity**”
* **How to build a lock?**
  * We would like to execute a series of instructions atomically, but due to the presence of interrupts on a single processor (or multi-thread on multi-processor), we couldn’t
  * We deal with this problem directly with the notion of lock
    * Programmers annotate source code with locks
    * Putting them around critical sections
    * And ensure that critical section executes as if it were a single atomic instruction
  * **#1: disabling interrupt for critical sections**
    * Steps
      * Code inside the critical section will not be interrupted, and will execute as if it were atomic
      * When we finish, reenable interrupts (via hardware instructions)
    * Pros: simplicity
    * Cons
      * Allow calling thread to perform privileged instructions and trust that it is not abused
      * On multi-processor, each threads running on different CPUs, and each try to enter the same critical sections, does not matter whether interrupts are disabled
  * **#2: Compare and Swap [hardware support]**
    * Basic Idea
      * Test whether the value at the address specified by `ptr` is equal to `expected`
      * If so, update the memory location pointed to by `ptr` with the new value
      * If not, do nothing
      * In either case, return original value at that memory location, allow the code calling this call knows it succeeds or not
    * Pros: more powerful instruction than test-and-set (i.e. in lock-free synchronization) setup

# Lock

* POSIX uses `mutex` as the name for lock
  * Stands for mutual exclusion between threads
  * If one thread is in the CR, excludes others from entering until it has completed the section

## Controlling Interrupts

* Earliest solution for single-processor system
```c
void lock() {
    disable_interrupts();
}
void unlock() {
    enable_interrupts();
}
```

* Pros: simplicity
* Cons
  * Requires us to allow any calling thread to perform privileged operation (turning interrupts on and off), and trust that this facility is but abused
    * Requires too much trust in applications
  * Does not work in multiprocessors
    * Multi-threads running on different CPU, and entering the critical section, it does not matter whether interrupts are disabled
  * Interrupts can be lost
    * I.e. CPU missed the fact that a disk device has finished a read request
  * Inefficient
    * Code that masks or unmasks interrupts tends to be executed slowly
* Pros: simplicity
* Cons
  * Allow calling thread to perform privileged instructions and trust that it is not abused
  * On multi-processor, each threads running on different CPUs, and each try to enter the same critical sections, does not matter whether interrupts are disabled

## Hardware support for locking: Test-And-Set (Atomic Exchange)

* Returns the old value pointed to by the old_ptr, simultaneously update said value to new
* This sequence of operation is done atomically
  * By making bot the test (of the old lock value)
  * And set (of the new value)
  * A single atomic operation, we ensure that only one thread acquires the lock
* Spin lock
  * Spins, using CPU cycles, until the lock becomes available
  * On a single processor: requires a **preemptive scheduler** (i.e. one that will interrupt a thread via a timer, in order to run different thread)
  * Evaluating spin lock
    * Correctness: mutual exclusion?
      * YES!
    * Fairness: how fair is a spin lock to a waiting thread?
      * NO FAIRNESS GUARANTEES
      * Might be starvation
    * Performance: what are the costs of using a spin lock?
      * Single CPU: performance is painful
        * Thread holding the lock is preempted within a critical section
        * Scheduler might run every other thread trying to acquire the lock
        * Each threads spin for duration of the time slice before giving up the CPU
      * Multi-CPU: works reasonably well if # of threads ~ # of CPUs

## Fetch-and-Add

* Atomically increments a value while returning the old value at a particular address
  * Can use this to build a ticket lock
* Difference
  * Ensures progress for all threads
  * Once a thread is assigned with a ticket value, will be scheduled in some point in the future

## Too much spinning: what now?

* Problem gets worse with more threads contending for a lock
* N - 1 time slice may be wasted in similar manner, spinning and waiting for a single thread to release the lock
* Problem: how can we develop a lock that doesn’t needlessly waste time spinning on the CPU?
  * **Hardware support alone cannot solve the problem**
  * **Need OS support too!**

## Using Queues: Sleeping Instead of Spinning

* OS support and a queue to keep track of which threads are waiting to acquire the lock
  * Queues to control who gets the lock next and thus avoid starvation
* `park()`: put a calling thread to sleep
  * `park()` is used to block the current thread. It puts the thread into a waiting state until it is explicitly unblocked.
  * The thread remains parked until it is unparked by another thread or it is interrupted.
  * This method does not release any locks held by the thread when it is called.
* `unpark()`: wake a particular thread
  * `unpark` is used to unblock a thread that was previously parked. It makes the specified thread eligible for scheduling.
  * If the specified thread is not currently parked, a future park call will return immediately.
  * `park` is typically used in scenarios where a thread needs to wait for a condition to be met or for some resource to become available. It's a way to efficiently block a thread without busy-waiting.
* park/unpark: Provides explicit control over thread execution. You can decide exactly when a thread should be suspended and resumed.
* yield: Relies on the thread scheduler to make decisions about which thread to run next. It’s more of a hint than a command.


## Others

* Futex from Linux
* Two phase lock
  * First phase: spin for a while, hope to acquire the lock
  * Second phase: caller put to sleep, only woken up when the lock becomes free

## Condition Variables

* **Condition variable**
  * Def: explicit queue that threads can put themselves on when some state of execution (i.e. some **condition**) is not as desired (by **waiting** on the condition)
  * Some other thread, when it changes said state, can then wake one (or more) of those waiting threads and allow them to continue (by **signaling** on condition)


# Condition Variable

## Problem: producer / consumer or bounded-buffer problem

* One or more producer threads, one or more consumer threads
* Producer that generates data and puts it into a buffer
* Consumer that takes data out of the buffer and processes it

### **Variables:**

* A mutex lock **`buffer_lock`** to protect the shared buffer.
* A condition variable **`not_full`** to signal that the buffer is not full.
* A condition variable **`not_empty`** to signal that the buffer is not empty.

### Pseudocode

Producer

```c
lock(buffer_lock)

while(buffer is full) {
    wait(buffer_lock, not_full)
}

add item to buffer
signal(not_empty)
unlock(buffer_lock)
```

Consumer

```c
lock(buffer_lock)
while(buffer is empty) {
    wait(buffer_lock, not_empty)
}
remove item from buffer
signal(not_full)
unlock(buffer_lock)
```


# Semaphore


* A semaphore: an object with an integer value that we can manipulate with two routines
  * `sem_wait()`
  * `sem_post()`

* From the 162 lecture
* Down() or P(): an atomic operation that waits for semaphore to become positive,
then decrements it by 1 
* Up() or V(): an atomic operation that increments the semaphore by 1, waking up a
waiting P, if any

```c
int sem_wait(sem_t *s) {
    decrement the value of semaphore s by one
    wait if value of semaphore s is negative
}

int sem_post(sem_t *s) {
    increment the value of semaphore s by one
    if there are one or more threads waiting on semaphore s, wake up one of them
}
```

* Locks are sometimes called a binary semaphore.

```c

// A binrary semaphore (i.e. a lock)
sem_t m;
sem_init(&m, 0, X); // initialize to X; what should X be?
sem_wait(&m);
// critical section here
sem_post(&m);

```

# Thread API

## How to create and control threads?

* `pthread_create`
  * Creates a new user-thread!
  * `thread`: pointer to a data structure `pthread_t`
  * `attr`: specify any attributes this thread might have, e.x. stack size, scheduling priority
  * `start_routine`: function pointer
  * `arg`: arguments to be passed
* `pthread_join`
  * `thread`: specify which thread to wait for
  * `value_ptr`: a pointer to the return value you expect to get back
* `pthread_mutex_lock`
* `pthread_mutex_unlock`
* Critical region.

### Signal

* Register a `SIGINT` signal handler
* For each signal, there is a default handler
  
```c
void signal_callback_handler(int signum) {
  printf(“Caught signal!\n”);
  exit(1);
}
int main() {
  struct sigaction sa;
  sa.sa_flags = 0;
  sigemptyset(&sa.sa_mask);
  sa.sa_handler = signal_callback_handler;
  sigaction(SIGINT, &sa, NULL);
  while (1) {}
}
```
* `SIGINT` - Control-C
* `SIGTERM` - default for `kill` shell command
* `SIGSTP` - Control-Z (stop process)

* Signalling must take place between threads.
* `pthread_cond_wait()`
  * puts the calling thread to sleep, and thus waits for some other thread to signal it, usually when something in the program has changed that the now-sleeping thread might care about

### Condition variable

* To use a condition variable, one has to in addition have a lock that is associated with this condition. When calling either of the above routines, this lock should be held.

```c
Pthread_mutex_lock(&lock);
while (ready == 0)
    Pthread_cond_wait(&cond, &lock);
Pthread_mutex_unlock(&lock);
```

* Acquire the Lock: The thread acquires the lock before checking the condition. This ensures that no other thread can change the condition while it is being checked.
* Wait on the Conditional Variable: The thread waits on the conditional variable, which atomically releases the lock and puts the thread to sleep. When the thread wakes up (due to a signal or broadcast), it re-acquires the lock before proceeding. This ensures the condition is re-checked in a protected manner.
* The code to wake a thread, which would run in some other thread:

```c
Pthread_mutex_lock(&lock);
ready = 1;
Pthread_cond_signal(&cond);
Pthread_mutex_unlock(&lock);
```

* First, when signaling (as well as when modifying the global variable ready), we always make sure to have the lock held. This ensures that we don’t accidentally introduce a race condition into our code. This ensures that waiting threads are properly synchronized with the state changes.
* However, before returning after being woken, the pthread cond wait() re-acquires the lock, thus ensuring that any time the waiting thread is running between the lock acquire at the beginning of the wait sequence, and the lock release at the end, it holds the lock.
* the waiting thread re-checks the condition in a while loop, instead of a simple if statement.
* Although it rechecks the condition (perhaps adding a little overhead), there are some pthread implementations that could spuriously wake up a waiting thread; in such a case, without rechecking, the waiting thread will continue thinking that the condition has changed even though it has not.

```c
while (ready == 0)
    ; // spin
```

* Use conditional variables. 

# Event-based Concurrency

## API: `select()` or `poll()`

* Check whether there is any incoming I/O that should be attended to
* I.e. a network application (web server) wishes to check whether any network packets have arrived, in order to service them
* Check whether descriptors can be read from as well as written to
  * Determine that a new packet has arrived and is in need of processing
  * Let the service know when it is ok to reply
* Timeout = NULL: block indefinitely until some descriptors is ready
  * Common: set it to 0, then use the call to `select()` to return immediately
* Poll() system is pretty similar
* Use non-blocking event loop

## Blocking v.s. Non-blocking Interfaces

* Blocking (or synchronous) interfaces do all of their work before returning to the caller
* Non-blocking (or asynchronous) interfaces begin some work but return immediately, thus letting whatever work that needs to be done get done in the background.
* Solution: Async I/O
  * `struct aiocb` or AIO control block
  * Application fill in this structure, then issue async call to read the file `int aio_read(struct aiocb *aiocbp);`
    * Try issuing I/O, if successful, return right away
  * `int aio_error(const struct aiocb *aiocbp);`
    * Checks whether the request referred to by `aiocbp` has completed
    * Periodically poll the system
    * Interrupt (signals) to inform application when AIO completes, removing needs to repeatedly ask the system

## Monitior

* A lock plus one or more condition variables
* – Always acquire lock before accessing shared data
* – Use condition variables to wait inside critical section
* - Three Operations: Wait(), Signal(), and Broadcast()