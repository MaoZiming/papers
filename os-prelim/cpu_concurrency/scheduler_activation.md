# Scheduler Activations: Effective Kernel Support for the User-Level Management of Parallelism (1991) 
The paper presents a mechanism called "scheduler activations" to improve the management of thread-level parallelism. It tries to address the limitations found in both user-level threads and kernel-level threads by offering a method for user-level thread libraries to communicate with the kernel scheduler. The paper outlines various techniques such as **upcalls** to communicate state information between the user and kernel levels and also discusses the notion of "virtual processors" provided by the kernel to the user-level threads for scheduling.

## V.s. user thread
Avoid unnecessary blocking and support processor allocation: 
* User-level thread system notifies kernel when it needs more or fewer processors (doesn’t happen very often)
* Kernel notifies the user-level thread system whenever kernel changes # of processors assigned or an I/O event occurs

## V.s. kernel thread 
Flexibility and performance: user-level thread system has control over which threads to run on the allocated v-processors, without trapping into the kernel. 

## V.s. Exokernel / Microkernel 
Exo-kernel: Like scheduler activations, exokernels aim to give more control to the application level for better resource management. Both try to minimize the kernel's role in making policy decisions, leaving that to the user level.

Microkernel: Scheduler activations share the microkernel's philosophy of minimizing kernel responsibilities and offloading more control to the user level. Microkernels, however, aim for complete modularity and separation of concerns, while scheduler activations work within the framework of a monolithic kernel to provide specific functionality.

## Limitations 
More complex to implement, upcall performance is limited, also there is not much performance analysis provided in the paper. 
