# Semaphore

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

- Locks are sometimes called a binary semaphore. 