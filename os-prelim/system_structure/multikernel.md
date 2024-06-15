# The Multikernel: A New OS Architecture for Scalable Multicore Systems (2009)
Reference: https://github.com/parasj/papers/blob/master/os_structures/multikernel.md

Increasing heterogenity and scalability in hardware (i.e. CPU cores) motivates modeling multicore systems as distributed systems. Three principles guide multikernel design 

(1) make all inter-core communication explicit via **message passing**

(2) make all OS structure **hardware-neutral** via separating OS structure from the hardware

(3) **replicate state** across cores instead of sharing state.

## Message passing v.s shared memory 
Ousterhout describes that core speed is increasing faster than disk and memory latency. So operating systems still block on memory and disk. Message passing is more efficient than shared memory at scale. So the multikernel embraces message passing (much like Mach).
