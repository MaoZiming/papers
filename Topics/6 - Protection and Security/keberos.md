# Kerberos: An Authentication Service for Open Network Systems (1998) 
Kerberos is an authentication network protocol developed by MIT to provide robust and **secure authentication system in an open network** where **users at workstations** wish to access **services on network servers**. Open networks are susceptible to various kinds of attacks, such as eavesdropping, replay attacks, and impersonation. 

The protocol is based on generating tickets 🎫.

## Goal
* Security: To protect sensitive data in open networks.
* Transparency: To facilitate straightforward authentication for users without the cumbersome need for repeated logins.

## Key Techniques: Ticket  
There are three components in Kerberos transactions: 
* **KDC (key distribution center)**
  *  Store a copy of all keys (i.e. client keys, service key, and TGS keys) for encryption and decryption purposes
  *  **Authentication server (AS)**: Validates users when they log in
  *  **Ticket granting server (TGS)**: Issues tickets for users to access different services
* **Client**: The user who wants to access a service.
* **Service**: The network resources like files servers or databases that the client wants to access.
 
## Protocol Simple Explain 
1. **Initial Authentication**:
    * Client sends login details to AS
    * AS replies with two messages
       * Ticket Granting Ticket (TGT) encrypted with TGS secret key
       * M1: TGS session key encrypted with client's secret key
2. **Request for Service Ticket**:
    * Decrypt M1 with secret key
    * Request access ot a service by taking to TGS
        * Send back TGT
        * Encrypt user authentication with TGS session key
        * M2: <service ID, request lifetime>
    * TGS receives
        * check service is valid
        * decrypt TGT with session key, and decrypt user authentication
    * TGS sends back
        * M3: Service session key encrypted by TGS session key
        * Service ticket (ST): encrypted by TGS secret key
3. **Accessing the service**:
    * User decrypt M3 and get servie session key
    * Use it to encrypt user authentication and send back service ticket
4. **Verification**
    * Service decrypt ST to get session key and decrypt user auth
    * If matches, send back service authenticator encrypted by TGS session key
    * Confirming authentication success and establish secure session

## Notes 
Trust is incrementally built through a series of encrypted ticket exchanges, and authentication is validated at several stages through encrypted, timestamped tickets

1. Mutual Trust via Third Party: The AS and TGS act as trusted third parties. The mutual trust between a user and a service is established indirectly via trust in these servers.

2. Scalability: Because users don't interact directly with service servers for authentication (but go through TGS instead), the system is more scalable. Each service doesn’t need to implement its authentication logic; it just needs to be able to decrypt a message from the TGS.
