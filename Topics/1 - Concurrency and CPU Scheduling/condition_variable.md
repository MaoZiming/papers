# Condition Variable

## Problem: producer / consumer or bounded-buffer problem

- One or more producer threads, one or more consumer threads
- Producer that generates data and puts it into a buffer
- Consumer that takes data out of the buffer and processes it

### **Variables:**

- A mutex lock **`buffer_lock`** to protect the shared buffer.
- A condition variable **`not_full`** to signal that the buffer is not full.
- A condition variable **`not_empty`** to signal that the buffer is not empty.

### **Producer:**

1. Lock **`buffer_lock`**.
2. While the buffer is full, wait on condition variable **`not_full`**.
3. Add the item to the buffer.
4. Signal the **`not_empty`** condition variable to indicate that the buffer now has at least one item.
5. Unlock **`buffer_lock`**.

### **Consumer:**

1. Lock **`buffer_lock`**.
2. While the buffer is empty, wait on condition variable **`not_empty`**.
3. Remove an item from the buffer.
4. Signal the **`not_full`** condition variable to indicate that the buffer now has at least one empty slot.
5. Unlock **`buffer_lock`**.

### Pseudocode ###

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