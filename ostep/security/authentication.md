# Authentication

- **Principal**
    - I.e. party asking for something
    - Security-meaningful entities that can request access to resources, such as groups of users, or complex software systems
    - Different OS have used different types of identity for principals
- **Agent**
    - I.e. the process or other active computing entity performing the request on behalf of a principal
    - The request is for accessing a particular resource, which is the **object** of access request
- **Credential**
    - I.e. any form of data created and managed by OS that keeps track of access decisions for future reference

### What you know

- Passwords and PIN
    - System stores the **hash** of the password, not the password itself
        - **Cryptographic hashes**: infeasible to use the hash to figure out what the password is
    - Not too careful with how you store the authentication information
    - By nature, can’t reverse hashing algorithms
    - **Dictionary attack:** try lists of names and words before random strings ****
        - Shut off access to an account when too many wrong guesses
        - Slow down process of password checking after a few wrong guesses
    - Attacker stole the password file?
        - Before hashing a new password and storing it in password file, generate a big random number and concatenate it into the password
        - Random number is called a **salt**
- **Multi-factor authentication**
    - **Two-factor authentication**
    - PIN and provide further evidence of your identity