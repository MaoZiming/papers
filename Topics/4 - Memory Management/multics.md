# Virtual Memory, Processes, and Sharing in MULTICS (1967) 
This paper presents the design of MULTICS, including its concepts of processes, address space, and virtual memory, and the use of paging and segmentation. It talks about how users may share procedures and data, and the mechanisms to dynamically transform symbolic references into VM address. 

This is basically the segmentation + paging techniques discussed in OSTEP. 

## Problem 
Before MULTICS there were no existing OS capable of providing users with a large machine-independent virtual memory. So the objectives of MULTICS are 
1. Provide user with ***large machine-independent virtual memory*** 
    1. I.e. responsibility of managing physical storage is now the system software 
2. Permit programming ***generality*** 
    1. One procedure can use another procedure only knowing its name, not the storage requirement 
3. Permit ***sharing of procedures and data*** among users subject only to proper authorization
