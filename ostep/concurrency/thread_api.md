# Thread API 

## How to create and control threads?

- `pthread_create`
  - `thread`: pointer to a data structure `pthread_t`
  - `attr`: specify any attributes this thread might have, e.x. stack size, scheduling priority
  - `start_routine`: function pointer
  - `arg`: arguments to be passed
- `pthread_join`
  - `thread`: specify which thread to wait for
  - `value_ptr`: a pointer to the return value you expect to get back
- `pthread_mutex_lock`
- `pthread_mutex_unlock`

### Condition variable
- Signalling must take place between threads.
- `pthread_cond_wait()`
  - puts the calling thread to sleep, and thus waits for some other thread to signal it, usually when something in the program has changed that the now-sleeping thread might care about
- To use a condition variable, one has to in addition have a lock that is associated with this condition. When calling either of the above routines, this lock should be held.

```c
Pthread_mutex_lock(&lock);
while (ready == 0)
    Pthread_cond_wait(&cond, &lock);
Pthread_mutex_unlock(&lock);
```

- Acquire the Lock: The thread acquires the lock before checking the condition. This ensures that no other thread can change the condition while it is being checked.
- Wait on the Conditional Variable: The thread waits on the conditional variable, which atomically releases the lock and puts the thread to sleep. When the thread wakes up (due to a signal or broadcast), it re-acquires the lock before proceeding. This ensures the condition is re-checked in a protected manner.


- The code to wake a thread, which would run in some other thread: 
```c
Pthread_mutex_lock(&lock);
ready = 1;
Pthread_cond_signal(&cond);
Pthread_mutex_unlock(&lock);
```
- First, when signaling (as well as when modifying the global variable ready), we always make sure to have the lock held. This ensures that we don’t accidentally intro- duce a race condition into our code. This ensures that waiting threads are properly synchronized with the state changes.
- However, before returning after being woken, the pthread cond wait() re-acquires the lock, thus ensuring that any time the waiting thread is running between the lock acquire at the beginning of the wait sequence, and the lock release at the end, it holds the lock.
- the waiting thread re-checks the condition in a while loop, instead of a simple if statement.
- Although it rechecks the condition (perhaps adding a little overhead), there are some pthread implementations that could spuriously wake up a waiting thread; in such a case, without rechecking, the waiting thread will continue thinking that the condition has changed even though it has not. 
  
```c
while (ready == 0)
    ; // spin
```
- First, it performs poorly in many cases (spinning for a long time just wastes CPU cycles). Second, it is error prone. As recent research shows [X+10], it is surprisingly easy to make mistakes when using flags (as above) to synchronize between threads; in that study, roughly half the uses of these ad hoc synchroniza- tions were buggy! Don’t be lazy; use condition variables even when you think you can get away without doing so.
  


