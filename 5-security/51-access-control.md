# Access Control

### ACL (Access Control List)

- **Each file** has its own ACL.
- **Resource-centric.**
- ACL is composed of
    - Owner of the file (stored in Inode)
    - Members of a particular group or users (stored in Inode)
    - Everybody else
- Some features along with ACL
    - Need to figure out where to store ACL near the file and dealing with search
    - In distributed environment, need to have a common view of identity across all machines for ACL to be effective
- Cons:
  - Figure out entire set of resources some person is permitted to access is expensive: require scanning every ACL in the system


### Capabilities

- A running process has some set of capabilities that specify its access permissions
- **Resource-centric**
- Capacities are bunch of bits
    - Specific to a resource
    - OS maintain its own protected capability list for each process
- Pros
    - Easy to determine which system resources a given principal can access
        - Look at the principal’s capability list
        - Revoking access is also easy
- Cons
    - Determine the entire set of principals who can access a resource become more expensive

## Mandatory and Discretionary Access Control

- **Discretionary access control**
    - Whether almost anyone or almost no one is given access to a resource is entirely at the discretion of the owning user
- **Mandatory access control**: more restrictive
    - At least some elements of the access control decisions in such systems are mandated by an authority, who can override the desires of the owner of the information
- Enhancements like **type enforcement** and **role-based access control** can make it easier to achieve the secure policy
    - Type enforcement: associates detailed access rules to particular subjects and objects.
      - E.g. Processes of type `web_t` can read files of type `doc_t`.
    - Role-based access control (RBAC)
        - Organizing access control on the basis of such roles and then assigning particular users to the roles they are allowed to perform makes implementation of many security policies easier

## Others

- Security failures due to faulty access control mechanisms are rare
- Security failures due to poorly designed policies implemented by those mechanisms are not

## Basics

- **Least privilege**: give a user or a process the minimum privileges required to perform the actions you want to show 
- **Least common mechanism**: use separate data structures or mechanisms for different users or processes (e.x. page table for each process) 
  - disallows the sharing of mechanisms that are common to more than one user or process if the users or processes are at different levels of privilege