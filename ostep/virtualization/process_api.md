# Process API

* Process ID (PID)
* for()
  * The value it returns to the caller of fork() is different. Specifically, while the parent receives the PID of the newly-created child, the child receives a return code of zero.
* exec() family of system calls allows a child to break free from its similarity to its parent and execute an entirely new program.
  * Loads the code (and static data) from that executable and overwrite its current code segment (and current static data) with it; the heap and stack and other parts of the memory space of the program are re-initialized 
  * Does not create a new process; transforms the currently running program into a different running program 
  * Never returns if success; as if the old program never runs. 
* wait()
  * Parent can wait for the child to finish executing, thereby finishing before the child.
* The separation of fork() and exec() enables features like input/output redirection, pipes and other cool features. 