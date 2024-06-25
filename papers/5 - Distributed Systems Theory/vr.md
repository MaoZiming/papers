# Viewstamp Replication 
Viewstamp Replication is a replication protocol developed in 1980's to handle failures in which nodes fail by crashing. 

## Goals of VR design 
1. _Availability_: user unaware of replicated service
2. _Consistency_: state machine replication
    - Clients can run general operations to observe and modify the service state
    - _Main Challenge_: ensure ops executed in the same order at all replicas despite failures


## Assumptions 
1. **Crash faults**: crash-stop or crash-recovery
2. **Partially synchronous**: message would eventually be delivered

## Protocol Basics 
### Replica Group 
Replica group of size **$2f+1$**
*   Able to carry request without $f$ replicas participating
*   But worst case, $f$ can just be slow
*   Need $f+1$ other replicas to ensure at least one executed request and didn't fail subsequently 

### Requirements 
1. Every op executed by the replica group survices into the future in spite of up to $f$ failures 
2. Handle concurrent client operation with some global order

### Approach 
* Primary: decide order for client requests
    *  also execute and return results to cleint after $f+1$ (self-includedd) replicas know about the request
* Client: monitor primary failures, initiate _view_ change if needed
    *  **view**: one of the replicas is selected to be the primary
    *  Goal (view-change): if primary fails, need to elect new replica to be primary 

_Reasoning_:
* Primary deciding order satisfieds requirement #2.
* Primary execute after hear back from $f+1$ ensures that even if $f$ replicas fail, the protocol continues to execute.
* However, primary itself can fail. So the view change protocol is for masking the failures of the primary. 

| Notion   |     Definition     | 
|----------| -------------|
| $c$ |  client-id | 
| $v$ |  view number | 
| $m$ |  msg received from client | 
| _op_ |  operation client wants to run | 
|_log_ | array of _op-numbers_, contain requests received so far in assigned order | 
| $n$ |  op-number assigned to request | 
| $i$ |  replica number | 
| $l$ |  replica log |

### #1: Normal Operation Protocol 
<img width="579" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/23dcbb8e-c1b2-48ee-ad19-f1405a15bafd">


1. Client sends <REQUEST _op_, $c$, $s$, $v$ > to primary
2. Primary advances _op-number_ and adds request to end of the _log_, then it sends < $PREPARE$, $m$, $v$, $n$ > to other replicas
3. Non-primary replicas process $PREPARE$ message in order, wait until it has all eariler requests in its _log_, then append and send < $PREPAREOK$, $v$, $n$, $i$ > to primary
4. Primary waits for $f$ $PREPAREOK$ message from different replicas, then it's considered as committed. Primary executes this request until all previous requests are executed and send < $REPLY$, $v$, $s$, $x$ > to the client where $x$ is the result
5. Primary inform other replicas about commit either through piggy-backed information or send directly, secondary then commits in order

### #2: View Change Protocol 
* The system moves through a sequence of views
* In each **view** one of the replicas is selected to be the primary
* View changes are used to mask failures of the primary
* The correctness condition: every committed operation survives into all subsequent views in the same position in the serial order
* View change protocol must obtain information from logs of at least $f+1$ replica
    * Ensure all **committed** operations will be known (i.e. recorded in at least one of these logs)
* Viewstamps: < _view-number_, _op-number_>   
    *  ensure operations that are "preparing" but not committed will not get lost
    *  i.e. a new leader may assign a previous used op-number to different operation, causing inconsistency
*  Once a node advances _view-number_ it no longer accepts messages from old view, instead it informs senders about new view 
 
Steps:
1. Replica $i$ suspects primary is faulty, advances its _view-number_, set its status to _view-change_, and send < $DOVIEWCHANGE$, $v$, $l$, $k$, $i$ > to the new primary
   *  Not accept any $PREPARE$ message from old primary 
2. When new primary receives $f+1$ messages
   *  Set new log = the _most recent_ of those messages (i.e. with the largest viewstamp).
   *  Set _op-number_ to latest entry in new _log_
   *  Change status to _normal_
   *  Inform other replicas completion of view change by sending < $STARTVIEW$ $v$, $l$, $k$ >
       *  $l$ is the new log and $k$ is the _op-number_ of latest committed request
3. The new primary execute (in order) any committed operations that it hadn't executed previously and send replies to clients, start accepting client request
4. Other replicas receiving $STARTVIEW$ will
   *  replace their log with the new log, set their _op-number_, _view-number_, and change their status to _normal_
   *  send $PREPAREOK$ messages for uncommitted ops 

### #3: Recovery Protocol 
1. Replica $i$ sends a < $RECOVERY$, $v$, $r$ > nessage to all other replicas, where $v$ is its starting _view-number_
2. Replica $i$ replies < $RECOVERY$ $RESPONSE$ $v$, $l$, $k$, $i$ > only when its status is $normal$ and its _view-number_ is greater than $v$, and is primary of its view.
3. Replica waits to receive response, then update its state. And send $PREPAREOK$ message for uncommitted ops. 
