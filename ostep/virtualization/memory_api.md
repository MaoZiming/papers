# Memory API

- Stack Memory
  - allocations and deallocations of it are managed implicitly by the compiler
  - Stack memory is sometimes called **automatic** memory
  - Local variable

- Heap Memory
  - Allocations and deallocations are explicitly handled by the programmers 
  - Following example 
    - `int *x = (int *) malloc(sizeof(int));`
    - Compiler knows to make room for a **pointer** to an integer when it sees `int *x`
    - When the program calls `malloc()`, it requests space for an integer on the heap
    - The routine returns the address of such an integer, which is then stored on the stack for use by the program 
- `malloc`
  - This invocation of `malloc()` uses the `sizeof()` operator to request the right amount of space; in C, this is generally thought of as a compile-time operator, meaning that the actual size is known at compile time and thus a number (in this case, 8, for a double) is substituted as the argument to `malloc()`.
- Common errors
  - Automatic memory management
    - A garbage collector runs and frees unused references
  - Errors
    - *Forget to allocate memory* —> segmentation fault
    - *Not allocating enough memory* —> buffer overflow
        > Even though it ran correctly once, doesn’t mean it’s correct
        > 
    - *Forgetting to initialize allocated memory*
        - Random
    - *Forgetting to free memory* —> memory leak
    - *Freeing memory before you are done* —> dangling pointer
    - *Freeing memory repeatedly* —> double free, undefined behavior
    - *Calling free() incorrectly* —> invalid frees
- malloc() and free(). The reason for this is sim- ple: they are not system calls, but rather library calls. 
  - `brk`, which is used to change the location of the program’s break: the location of the end of the heap.
  - `sbrk`
  - By passing in the correct arguments, `mmap()` can create an anonymous memory region within your program — a region which is not associated with any particular file but rather with swap space,