# Distributed System Security 

- Symmetric crypto: used for transport of most data, since it is cheaper
    - Not shared by system participants before the communication starts
    - First step in protocol: exchange symmetric keys
    - Diffie-Hellman key exchange is commonly used

## SSL / TLS: a protocol, not a software package

- Standard method of communicating between processes in modern system is socket
- Secure Socket Layer (SSL)
    - Formally is insecure
    - Move encrypted data through an ordinary socket
    - I.e. set up a socket, set up a special structure to perform crypto, then hook the output of that structure to the input of the socket, reverse the process on the other end
    - Steps
        - Start negotiation between client and server
        - End in both sides finding some acceptable set of ciphers and techniques that balance between performance and security
            - I.e. Diffie-Hellman key exchange to create the key
- Transport Layer Security (TLS)
    - Use this
    - Evolved from SSL.
    - It should be noted that TLS does not secure data on end systems. It simply ensures the secure delivery of data over the Internet, avoiding possible eavesdropping and/or alteration of the content.
    - TLS is normally implemented on top of TCP in order to encrypt Application Layer protocols such as HTTP, FTP, SMTP and IMAP, although it can also be implemented on UDP, DCCP and SCTP as well 
    - However, SSL is an older technology that contains some security flaws. Transport Layer Security (TLS) is the upgraded version of SSL that fixes existing SSL vulnerabilities.