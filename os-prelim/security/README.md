# Security 
* Reference: https://github.com/parasj/papers/tree/master/security 

Security flaws in OS have immense impact on many users and pieces of software. All of the software relies on proper behavior of underlying hardware, OS has ultimate control of hardware resources. In OS, lots of components can go wrong; with multi-process, the security question becomes even harder. 

How to ensure that the resources each process is permitted to access are exactly those it should access in ways we desired? What primitives are needed from the OS? What mechanisms should be provided from the hardware? 

## Security Guideline
### Authentication  
* How to attenticate users
  *  What you know: passwords and pins (crypto), multi-factor authentication
  *  What you have: tickets, IDs, hardware token
  *  What you are: physical characteristics, DNA, face, etc.  
### Access control
* Generally spoken in terms of subjects, objects, and access
  *  subject: entity that wants to perform the access
  *  object: thing the subject wants to access
* AC: whether a subject is allowed to perform a particular mode of access on a particular object
* Basic approaches
  *  Access control list (ACL) (i.e. per file) 
  *  Capabilities (i.e. like keys)
  
### Flow control 
* Prevent information from being transmitted from one part of the system to another. Idea behind HiStar
* Data has a particular level of security and cannot be lowered
* Data can become overclassified as programs may interact with data across classes
* Covert channels are hard to stop
* Have to be careful to prevent people from copying data

### Crypto control 
* Crux: how to protect information outside the OS’s domain 
* Use crypto to ensure that even others gain access to critical data, they are not able to use it
*  Distributed system crypto
  *  Messages between components are encrypted and authenticated
  *  Symmetric v.s asymmetric crypto
  *  SSL / TLS and public keys  
