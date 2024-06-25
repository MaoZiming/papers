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
  * The separation of fork() and exec() is essential in building a UNIX shell, because it lets the shell run code after the call to fork() but before the call to exec(); this code can alter the environment of the about-to-be-run program, and thus enables a variety of interesting features to be readily built.
  * The way the shell accomplishes input/output redirection is quite simple: when the child is created, before calling exec(), the shell (specifi- cally, the code executed in the child process) closes standard output and opens the file newfile.txt. By doing so, any output from the soon- to-be-running program wc is sent to the file instead of the screen
* Shell:
  * It shows you a prompt and then waits for you to type something into it. You then type a command (i.e., the name of an executable program, plus any arguments) into it; in most cases, the shell then figures out where in the file system the executable resides, calls fork() to create a new child process to run the command, calls some variant of exec() to run the command, and then waits for the command to complete by calling wait(). When the child completes, the shell returns from wait() and prints out a prompt again, ready for your next command
* Signals
  * Process control is available in the form of signals, which can cause jobs to stop, continue, or even terminate.
  * a process should use the signal() system call to “catch” various signals; doing so ensures that when a particular signal is deliv- ered to a process, it will suspend its normal execution and run a particu- lar piece of code in response to the signal. 