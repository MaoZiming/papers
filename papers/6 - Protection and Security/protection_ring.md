# A Hardware Architecture for Implementing Protection Rings (1972) 

The paper discusses protection of segment-based virtual memory as found in Multics. Read, write, and execute permissions are set based on the access level of the process and recorded in segment descriptor words. Flags corresponding to these permissions are checked by hardware upon each reference. 

Reference: https://github.com/parasj/papers/blob/master/security/protection_rings.md

* Protection rings are nuanced protection modes beyond user/supervisor
* Ring 0 is most priviledged
* Multics segment descriptors indicate the highest ring that is allowed to read or write the segment
* Rings with less privilege of the system do not have execute permissions 

## Protection rings 
Hierarchical levels of security that help to define the privileges and access rights that different processes or programs have when interacting with the system. 

Or, a strategy in computer architecture to govern how different programs can interact with the system's resources, helping to enhance security and system stability.

### Implementation in this paper 
Recorded in segment descriptor words and are checked by the hardware upon each reference. 
