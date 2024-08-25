# UNIX Implementation (1978) 

* **Key idea:** File-system interface that enables use by multiple users, a simplified device interface, and a user-shell with composable programs
* Key features:
  1) File-system (interface to persistent storage) 
     * Ordinary files, directories and `/dev`
       * `/dev`: read and write to IO devices
       * Advantages: (1) similar to file IO, (2) can pass device to a program, (3) can protect devices with file permissions
     * Mount/Unmount filesystems
     * Permissions
       * The `setuid` bit is used to temporarily elevate privileges when executing a program.
       * super-user has permissions for all
     * No filesystem locks (unecessary and insufficient)
  2) Processes
     * text, data and then stack segments
     * pipes and file descriptors abstract communication (like file IO)
  3) Shell
     * Can call programs from a shell
     * Pipes to connect programs, and filter
     * Backgrounding
  4) Traps
* Lessons
  * Simplicity is key; size constraint leads to simplicity as the design. 
  * Internal requirements led to good design, rather than any particular customer
  * Designed for programmers, which makes it capable for more users as an interactive system (versus batch)
* Use staleless signals to indicate which resources are free as opposed to stateful semaphores. 
* Introducing fork and exec. 
