# Introduction

- Three Easy Pieces:
  - Virtualization
  - Concurrency
  - Persistence
- Processor (Von Neumann model of computing):
  - Fetch an instruction from memory
  - Decode the instruction (Figure out what instruction it is)
  - Execute the instruction

- OS
  - a body of software that is in charge of making sure the system operates correctly and efficiently in an easy-to-use manner 
  - Exports hundreds of system calls to applications.
  - OS provides a standard library to applications. 

- Virtualization
    - Concept: takes a physical resource (i.e. processor, memory, disk) and transforms it into a more general, powerful, and easy-to-use virtual form of itself
    - E.g. OS is virtualizing memory (0x200000): each process accesses its own private virtual address space. 
- History
  - Early: just libraries
    - Batch processing: # of jobs were set up and run in a “batch” by the operator
    - Not interactive, too expensive (thousands of dolloars per hour)
  - Beyond: protection 
    - system call
      - file system as a library makes little sense for the notion of privacy; any program can read any file.
      - Instead of providing OS routines as a library (where you just make a procedure call to access them), add a special pair of hardware instructions and hardware state to make the transition into the OS a more formal, controlled process
    - Difference between a system call and a procedure call: a system call transfers control (i.e., jumps) into the OS while simultaneously raising the hardware privilege level
      - User applications run in user mode. 
      - trap: initiate system call, a special hardware instruction.
      - hardware transfers control to a pre-specified trap handler
      - raises privilege level to kernel mode (full access to HW, i.e. I/O request, make more memory available)
      - return-from-trap instruction to revert to user mode 
  - The era of multiprogramming 
    - concurrency becomes an issue 
    - UNIX
  - The modern era 
    - PC, low cost 