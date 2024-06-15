# Practical Byzantine Fault Tolerance (1999) 
PBFT is **efficient** replication protocol extended from Viewstamp Replication that allows the group to survice **Byzantine (arbitrary) failures**. By this time, there was a realization that malicious attacks and Byzantine behavior needed to be dealt with.

## Key Motivation 
Traditional replication protocol (e.x. Paxos, Raft, VR) does not survive Byzantine failures, which are more commonly seen back in the time in forms of mailicious attac, software errors, and so on. 

Exsiting BFT algorithms assume synchrony for safety, but this is dangerous since a malicious attacker can delay messages through DoS, and also they are inefficient. 

## Setup 
### System model 
* Byzantine faults: allow replicas to behave arbitrarily (e.x. not reply, reply bad, accept request but discard state)
* Partial synchrony model: message eventually delivered

### Replica Group 
* Up to $f$ replicas are faulty
* Use $3f+1$ replica group
    *  $N$ replicas, tolerate $f$ non-replies
    *  Then get $N - f$ replicas, worst-case contains $f$ Byzantine faulty replicas
    *  $N - f - f$ none-faulty nodes needs to be larger than $f$ faulty nodes to reach agreement
    *  So $N - 2f > f$, we get $N$ is at least $3f+1$

### Architecture
* Primary: order client request
    * execute and reply
* Client: monitor primary, do view-change if necessary
* Difference v.s VR
    *  #1: Replicas are Byzantine
    *  #2: Adversary can control the network: remove messages, corrupt them, etc.
*  Approach     
    *  #1: 3-phase protocol to tolerate $f$ byzantine
    *  #2: use cryptography - all msgs signed by senders
        *  goal: ensure secret keys used by honest is not known to adversay

### Security techniques
Goal: Prevent spoofing and elays and corrupting messages

Techniques 
*   Public-key signature: one cannot impersonate other
      *   E.x. messages between clients and the primary 
*   Message authentication code, collision-resistent hash, digest: one cannot tamper other's meesages
      *   Digest: crytographic hash of a piece of data, reduce amount of data to be sent
      *   Collision-resistent hash: infeasible to find two different inputs that produce the same hash output 
 
## Protocol: trust group not individuals 

### Normal Operation Protocol 
<img width="741" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/5027562f-ddf6-4de7-b454-15a2b29df3f2">


<img width="640" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/86fc72cc-3564-4222-88a0-28036e0f7b4b">


* Also, client only accepts a result if it receives $f+1$ authenticated messages with the same result 
* Why pre-prepare stage? 
      *   Basic idea: Replica can be faulty! (i.e. assign wrong timestamp, etc.)

### View Change Protocol 
<img width="744" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/ff625fd6-e167-4fcb-9fff-5b79dfafeada">

*   _certificate_: a collection of matching valid signed messages from $2f+1$ different replicas, representing **a proof that certain thing has happened**
      *   messages are signed, so replica is able to evaluate a certificate and decide for itself whether it is valid    
*  Need to use PREPARE certificates (i.e. composed of the messages replicas received while running the protocol)
      *  Consists of the PREPREPARE message from primary and $2f$ PREPARE message all for the same request( i.e. represented as the message digest) from the same viewstamp. 

* Certificate
      *   Used Step 1: a replica can reliably inform the primary about the requests that have prepared at it
      *   Used Step 2: consists of the $2f+1$ DOVIEWCHANGE messages; these allow the other replicas to construct a valid log and to check that the set O is correct. Note 
### Limitations 
Drawback: all-to-all communication with $O(n^2)$ 
