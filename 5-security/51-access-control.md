# Access Control

### ACL (Access Control List)

* **Each file** has its own ACL.
* **Resource-centric.**
* ACL is composed of
    * Owner of the file (stored in Inode)
    * Members of a particular group or users (stored in Inode)
    * Everybody else
* Some features along with ACL
    * Need to figure out where to store ACL near the file and dealing with search
    * In distributed environment, need to have a common view of identity across all machines for ACL to be effective
* Cons:
  * Figure out entire set of resources some person is permitted to access is expensive: require scanning every ACL in the system


### Capabilities

* A running process has some set of capabilities that specify its access permissions
* **Resource-centric**
* Capacities are bunch of bits
    * Specific to a resource
    * OS maintain its own protected capability list for each process
* Pros
    * Easy to determine which system resources a given principal can access
        * Look at the principal’s capability list
        * Revoking access is also easy
* Cons
    * Determine the entire set of principals who can access a resource become more expensive

## Mandatory and Discretionary Access Control

* **Discretionary access control**
    * Whether almost anyone or almost no one is given access to a resource is entirely at the discretion of the owning user
* **Mandatory access control**: more restrictive
    * At least some elements of the access control decisions in such systems are mandated by an authority, who can override the desires of the owner of the information
* Enhancements like **type enforcement** and **role-based access control** can make it easier to achieve the secure policy
    * Type enforcement: associates detailed access rules to particular subjects and objects.
      * E.g. Processes of type `web_t` can read files of type `doc_t`.
    * Role-based access control (RBAC)
        * Organizing access control on the basis of such roles and then assigning particular users to the roles they are allowed to perform makes implementation of many security policies easier

## PKI: Public Key Infrastructure

* Specifically, the most common form of encryption used today involves a public key, which anyone can use to encrypt a message, and a private key (also known as a secret key), which only one person should be able to use to decrypt those messages.
* Common examples of PKI security today are SSL certificates on websites so that site visitors know they’re sending information to the intended recipient, digital signatures, and authentication for Internet of Things devices.

## Symmetric vs. asymmetric encryption

* Symmetric: encrypt and decrypt with the same key. Think about imitation game.
* Asymmetric: Involving two different keys: a private key and a public key. 
  * Public key to encrypt, private key to decrypt. 

## Others

* Security failures due to faulty access control mechanisms are rare
* Security failures due to poorly designed policies implemented by those mechanisms are not

## Basics

* **Least privilege**: give a user or a process the minimum privileges required to perform the actions you want to show 
* **Least common mechanism**: use separate data structures or mechanisms for different users or processes (e.x. page table for each process) 
  * disallows the sharing of mechanisms that are common to more than one user or process if the users or processes are at different levels of privilege

### Security

* **Cryptography** achieves protection by converting data’s original bit pattern into a different bit pattern, using an algorithm called cipher
* **Symmetric key encryption** (ciphers): using a single secret key shared by all parties with rights to access the data
    * Pros: simplicity, speedy
    * Cons: key distribution problem (i.e. if someone intercepts the key during transmission, whole system compromise), doesn’t provide **non-repudiation** (i.e. the ability to prove that a sender is the true sender)
* **Public-key encryption**: have two different keys for cryptography, one to encrypt and one to decrypt, with one keys kept secret and the other commonly made public
    * Pros: solve the key distribution problem, secure key exchange
    * Cons: speed, complexity
* **Cryptographic hashes**: special category of hash function with important properties
    * Computationally infeasible to find two inputs that will produce the same hash value
    * Any change to input will result in unpredictable change to resulting hash value
    * Computationally infeasible to infer any properties of the input based on the hash value
    * Note: no key, no one should be able to obtain the original bit patterns from the hash
    * Then, care about data integrity?
        * Take crypto hash of the data, encrypt only that hash, send both the encrypted hash and unencrypted data to partner
        * If opponent fiddles with the data in transit, decrypt the hash and repeating the hashing operation on data, find mismatch

### Digital Signatures

* Digital signature is one example of encrypting with private key, decrypting with public key. It is different from Public Key Infrastructure, where you encrypt with public key and decrypt with private key.

**Digital signatures** are a cryptographic technique that ensures the authenticity and integrity of a message or file. Here's how they work:

1. **Key Pair Generation**: 
   - A user generates a pair of cryptographic keys: a private key and a public key.
   - The private key is kept secret, while the public key is shared with others.

2. **Signing a File**:
   - When a file is created or modified, the owner uses their private key to generate a digital signature for the file.
   - This is done by creating a hash of the file (using a cryptographic hash function like SHA-256) and then encrypting the hash with the private key. The result is the digital signature.
   - The digital signature is unique to both the file and the private key.

3. **Verifying a File**:
   - When someone else wants to verify the file, they take the file and the digital signature.
   - They use the same cryptographic hash function to generate a hash of the file.
   - Then, they decrypt the digital signature using the public key to retrieve the original hash that was signed.
   - If the newly computed hash matches the hash retrieved from the signature, the file is authentic and has not been tampered with. If they do not match, the file has either been altered or was not signed by the holder of the private key.

**Key Benefits**:
- **Integrity**: Ensures the file has not been modified.
- **Authenticity**: Confirms that the file was signed by the legitimate owner of the private key.

### Merkle Trees

**Merkle Trees** are a data structure used to efficiently and securely verify the integrity of large datasets or collections of files. Here's how they work:

1. **Structure**:
   - A Merkle tree is a binary tree where each leaf node represents the hash of a data block (e.g., a file).
   - Each non-leaf (intermediate) node is the hash of the concatenation of its two child nodes' hashes.
   - The root node, known as the **Merkle root**, is a hash representing the entire dataset.

2. **Creating a Merkle Tree**:
   - Start by hashing each file or data block to create the leaf nodes.
   - Pair up the leaf nodes and hash each pair to create the parent nodes.
   - Repeat this process up the tree until a single hash (the Merkle root) is obtained at the top.

3. **Verifying Data**:
   - To verify a specific file, you only need the file's hash and the hashes of its sibling nodes up to the root.
   - You can recompute the necessary parent hashes and compare the resulting hash with the stored Merkle root.
   - If they match, the file is part of the original dataset and has not been tampered with.

**Key Benefits**:
- **Efficiency**: You can verify the integrity of a specific file without needing to rehash the entire dataset, just the path from the file to the root.
- **Scalability**: Works well with large datasets since the size of the proof (the number of hashes needed for verification) is logarithmic in relation to the number of files.

## SSL/TLS Overview

### What is SSL/TLS?

**SSL (Secure Sockets Layer)** and **TLS (Transport Layer Security)** are cryptographic protocols designed to provide secure communication over a computer network. SSL is the predecessor of TLS; TLS is an improved and more secure version of SSL.

### How Does SSL/TLS Work?

1. **Handshake Process**:
    - When a client (e.g., a web browser) connects to a server (e.g., a website), the SSL/TLS handshake begins.
    - The client and server agree on the encryption methods and protocols to use.
    - The server sends its SSL/TLS certificate, which includes its public key.
    - The client verifies the certificate's authenticity against a trusted Certificate Authority **(CA)**.
    - If verified, the client generates a session key, encrypts it with the server's public key, and sends it to the server.
    - The server decrypts the session key using its private key, establishing a secure encrypted session.

2. **Data Encryption**:
    - Once the handshake is complete, the session key is used to encrypt all data exchanged between the client and the server.
    - This ensures that any data transmitted is secure and cannot be intercepted or tampered with.

### Key Concepts

- **Certificates**: Digital documents issued by a Certificate Authority (CA) that authenticate the identity of the server.
- **Public/Private Key Pair**: Used during the handshake to encrypt and decrypt the session key.
- **Symmetric Encryption**: Used after the handshake to encrypt the communication using the session key.

### Importance of SSL/TLS

- **Confidentiality**: Data is encrypted, ensuring that only the intended recipient can read it.
- **Integrity**: Ensures that the data has not been altered during transmission.
- **Authentication**: Verifies the identity of the parties involved in the communication.

### Common Uses of SSL/TLS

- **HTTPS**: The most common use of SSL/TLS is in securing HTTP traffic, resulting in HTTPS (HTTP Secure).
- **Email**: Securing email communication using protocols like SMTPS, IMAPS, and POP3S.
- **VPNs**: Virtual Private Networks (VPNs) use SSL/TLS to secure the connection between the client and the VPN server.

### Conclusion

SSL/TLS is essential for ensuring the **security and privacy of online communications**. With the continuous evolution of security threats, TLS has become the standard for secure communication, replacing the older SSL protocol.
