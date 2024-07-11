# Making Information Flow Explicit in HiStar (2006)  

Link: http://www.scs.stanford.edu/~nickolai/papers/zeldovich-histar.pdf

Read: July 11th, 2024.

Secure OS by **restricting information flow**. Private user data is marked as "tainted" and thus is restricted from leaving the computer. Thus data cannot leak from the computer. Only a small core of code need be verified.

Currently OSes have too many aspects that need to be secured (i.e. sloppy code in many places lead to vulnerabilities). HiStar offers a minimalistic kernel that exposes information flow. 

There are six objects in the kernel, including: 
* Segment
* Thread
* Address space
* Device (network)
* Gate (IPC)
* Container ("directory"). Each object has a label (categories / colors), information flow controlled between objects. 
  * Think of the label as tainted bits.
* **Tainted processes can only talk to other tainted processes.**
* Threads can *taint* themselves to read tainted files. 

All of Unix implemented on top of these few abstractions, and new applications can implement security using these abstractions. 

High-level idea: 
* Subjects and Objects given a level, e.g. 1...4 (unclassified...top secret)
* No read up:
  * Subject at level i cannot read object at level j when i < j
* No write down:
  * Subject at level i cannot write object at level j when i > j
* Unix channels are made explicit, in terms of the HiStar mechanism.

## Main Idea
The main idea is to use **a small kernel that controls information flow**: explicit access checks on information flows can prevent data leaks. For example if AV code is malicious and it can communicate code over Internet, the kernel can simply not allow AV to send info anywhere. 

To track information flow, HiStar associate a label with the data, where label follows data when it moves around and determine what you can do with the data. HiStar will only allow kernel objects to interact (information to flow) if two kernel obejcts have "**consistent**" labels. 

### Scenario 3: Designing a System with Explicit Information Flow

### Question:

You're tasked with developing an OS where making information flow explicit is a key goal. How would you design the system to achieve this?

### Sample Answer:

Taking inspiration from HiStar, I would use data labeling to make information flows explicit. Every piece of data would have a security label, and the OS would enforce rules based on these labels to ensure secure data handling. In addition to this, I would leverage secure enclaves to protect the confidentiality and integrity of sensitive data. This way, even if an application is compromised, the secure enclave ensures that data governed by strict flow policies remains secure.
