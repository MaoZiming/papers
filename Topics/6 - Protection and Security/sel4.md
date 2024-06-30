# seL4: Formal Verification of an OS Kernel (2009) 

Link: http://web1.cs.columbia.edu/~junfeng/09fa-e6998/papers/sel4.pdf

June 29th, 2024. 

This paper presents the experience in performing the formal, machine-checked verification of the seL4 microkernel from an abstract specification down to its C implementation. This is the first formal proof of functional correctness of a complete, general-purpose operating-system kernel; assuming the correctness of the compiler, assembly code, boot code, managements of caches, and the hardware. 

The authors implement the OS first in Haskell to prototype proofs and refine the spec. Then, they implemented it in C and then used the spec to verify their C implementation. This enables high performance while ensuring correctness. Some parts of the verification steps seem to be manual. But it seems to ensure confidence that correctness is preserved.

<img width="604" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/04620d93-d741-47b7-8488-78996ec8d922">

### ummary

> Complete formal verification is the only known way to guarantee that a system is free of programming errors
> 

This paper presents the experience in performing the formal, machine-checked verification of the seL4 microkernel from an abstract specification down to its C implementation. This is the first formal proof of functional correctness of a complete, general-purpose operating-system kernel; assuming the correctness of the compiler, assembly code, boot code, managements of caches, and the hardware.

> We have created a methodology for rapid kernel de- sign and implementation that is a fusion of traditional operating systems and formal methods techniques. We found that our verification focus improved the design and was surprisingly often not in conflict with achieving performance.

### Verification

> The central artefact is the Haskell proto- type of the kernel. The prototype requires the design and implementation of algorithms that manage the low-level hardware details. To execute the Haskell prototype in a near-to-realistic setting, we link it with software (derived from QEMU) that simulates the hardware platform. Normal user-level execution is enabled by the simulator, while traps are passed to the kernel model which computes the result of the trap. The prototype modifies the user-level state of the simulator to appear as if a real kernel had executed in privileged mode.

Subset of language Huskell that can be translated into the language of the theorem prover we use. 

Translation from Huskell to C as the Huskell runtime is bulky. 

### Insights

- Microkernels are so small so that they can be formally verified!

### Techniques

Kernel Design Process 

- OS developers tend to take a bottom-up approach to kernel design
- Formal methods practitioners tend toward top-down design
- Use functional programming language Haskell to provide a programming language for OS developers, while at the same time can be translated into theorem proving tool
- Re-implement it in C
    - Haskell runtime is significant body of code, relies on GC that is unsuitable for real-time environments

- Abstract specification
    - Isabelle / HOL (Higher Order Logic) code
    - Describes what the system does without saying how it is done, like functional behavior of kernel operations
    - Example: scheduling, no scheduler policy is defined at abstract level
    - Make use of non-determinism in order to leave implementation choices to lower levels

- Executable specification
    - Haskel
    - Generated from Haskell into the theorem prover and fills in the details left open at the abstract level, and to specify how the kernel works as opposed to what it does
    - Deterministic; the only non-determinism left is the underlying machine
- C implementation
    - The most detailed layer in the verification
    - The translation from C into Isabelle is correctness-critical and model the semantics of C subset precisely and foundationally
- Machine model
    - Programming in C is not sufficient for implementing a kernel, like assembly
    - The basis of this formal model of the machine is the internal state of the relevant devices, collected in one record machine_state

Proof 

Use a refinement proof. 
> A refinement proof establishes a correspondence between a high-level (abstract) and a low-level (concrete, or refined) representation of a system.

- Prove by showing formal refinement
- To show that a concrete state machine $M_2$ refines an abstract one $M_1$, it is sufficient to show that for each transition in $M_2$ that may lead from an initial state $s$  to a set of states $s',$ there exists a corresponding transition on the abstract side from an abstract state $\sigma$ to a set $\sigma'$
- The transitions correspond if there exists a relation $R$   between the states $s$ and $\sigma$ such that for each concrete state in $s'$ there is an abstract one in $\sigma'$ that makes $R$   hold between them again
- Let
    - machine $M_A$ denote the system framework instantiated with the abstract specification
    - machine $M_E$  represent the framework instantiated with the executable specification
    - machine $M_c$  stand for the framework instantiated with the C program read into the theorem prover
- Theorem 1: $M_E$ refines $M_A$
- Theorem 2: $M_C$ refines $M_E$
- Theorem 3: $M_C$  refines $M_A$

### Limitations

- I think I don’t understand this paper to be honest…

### Questions

- Modularity design: change to global invariants and re-verify stuff
    - Verify components
    - Verify exception handlings: meet the specification v.s OS meeting the specification
    - Wrong behavior instead of crashing
    - Just verifying the error handling
- Alternative approach to build a safe kernel: high-level language
    - Runtime checks and other features
    - Write in goal: https://www.usenix.org/conference/osdi18/presentation/cutler
    - Problem we can solve for
        - Similar to type safety
        - The language can solve subset of the problem
        - More about: programmability rather than safety
- Modular approach, make changes, easier to iterate through the changes
    - Microkernel is setup to be more modular
