# Making Information Flow Explicit in HiStar (2006)  
Secure OS by **restricting information flow**. Private user data is marked as "tainted" and thus is restricted from leaving the computer. Thus data cannot leak from the computer. Only a small core of code need be verified.

Currently OSes have too many aspects that need to be secured (i.e. sloppy code in many places lead to vulnerabilities). HiStar offers a minimalistic kernel that exposes information flow. 

There are six objects in the kernel, including segment, thread, address space, device (network), gate (IPC), container ("directory"). Each object has a label (categories / colors), information flow controlled between objects. 

All of Unix implemented on top of these few abstractions, and new applications can implement security using these abstractions. 

## Main Idea
The main idea is to use **a small kernel that controls information flow**: explicit access checks on information flows can prevent data leaks. For example if AV code is malicious and it can communicate code over Internet, the kernel can simply not allow AV to send info anywhere. 

To track information flow, HiStar associate a label with the data, where label follows data when it moves around and determine what you can do with the data. HiStar will only allow kernel objects to interact (information to flow) if two kernel obejcts have "consistent" labels. 
