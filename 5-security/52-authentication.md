# Authentication

* **Principal**
    * A principal in computer security is an entity that can be authenticated by a computer system or network.
      * Security-meaningful entities that can request access to resources, such as groups of users, or complex software systems
    * Different OS have used different types of identity for principals
* **Agent**
    * I.e. the process or other active computing entity performing the request on behalf of a principal
    * The request is for accessing a particular resource, which is the **object** of access request
* **Credential**
    * I.e. any form of data created and managed by OS that keeps track of access decisions for future reference

### What you know

* Passwords and PIN
    * System stores the **hash** of the password, not the password itself
        * **Cryptographic hashes**: infeasible to use the hash to figure out what the password is
    * By nature, can’t reverse hashing algorithms
    * **Dictionary attack:** try lists of names and words before random strings
        * Shut off access to an account when too many wrong guesses
        * Slow down process of password checking after a few wrong guesses
    * **Salt**
        * Before hashing a new password and storing it in password file, generate a big random number and concatenate it into the password
        * Random number is called a **salt**
          * Salt is mainly to prevent hash collision. Since each user's password is salted with a unique value, the attacker has to compromise each password individually.
* **Multi-factor authentication**
    * **Two-factor authentication**
    * PIN and provide further evidence of your identity


### Scenario 1: Designing a Secure Authentication Mechanism for a Distributed System

* Question:
You're tasked with designing a secure authentication mechanism for a distributed system. Can you walk me through how you would accomplish this, and what technologies or methodologies you would leverage?

### Sample Answer:

I would opt for a Kerberos-like architecture to provide strong authentication. A central Authentication Server (AS) and a Ticket Granting Server (TGS) would be responsible for issuing time-bound tickets. This prevents replay attacks, following Kerberos principles. In addition to that, I would leverage secure enclaves, like Intel SGX, to ensure that sensitive authentication data is processed in a secure, isolated environment. This would add an additional layer of hardware-based security, much like the protection rings suggested by Schroeder and Saltzer, but with modern capabilities.


### Scenario 4: Combining Multiple Security Paradigms

### Question:

* Given your expertise in various security paradigms like protection rings, Kerberos authentication, and explicit information flow, how would you design a system that incorporates all of these?

### Sample Answer:

To combine these paradigms, I'd implement a multi-layered approach:

* **Authentication**: Use a Kerberos-like protocol for strong, time-bound authentication, reducing the risk of replay attacks.
* **Hardware-level Protection**: Utilize hardware features like protection rings or secure enclaves to offer additional layers of security. Secure enclaves would be particularly useful for executing sensitive tasks.
* **Information Flow**: Implement HiStar-inspired labeling for data and tasks to control their flow within the system explicitly, ensuring that even if one layer is compromised, others remain secure.

By integrating these paradigms, we can construct a system that is robust against a wide range of attack vectors.