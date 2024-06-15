# seL4: Formal Verification of an OS Kernel (2009) 
This paper presents the experience in performing the formal, machine-checked verification of the seL4 microkernel from an abstract specification down to its C implementation. This is the first formal proof of functional correctness of a complete, general-purpose operating-system kernel; assuming the correctness of the compiler, assembly code, boot code, managements of caches, and the hardware. 

The authors implement the OS first in Haskell to prototype proofs and refine the spec. Then, they implemented it in C and then used the spec to verify their C implementation. This enables high performance while ensuring correctness. Some parts of the verification steps seem to be manual. But it seems to ensure confidence that correctness is preserved.

<img width="604" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/04620d93-d741-47b7-8488-78996ec8d922">
