# Access Control

### ACL

- Each file has its own ACL
    - Looks up to perform access control
- ACL composes of
    - Owner of the file (stored in Inode)
    - Members of a particular group or users (stored in Inode)
    - Everybody else
- This saves the cost of accessing and checking them
- Some features along with ACL
    - Need to figure out where to store ACL near the file and dealing with search
    - Figure out entire set of resources some principal is permitted to access requires scanning every ACL in the system
    - In distributed environment, need to have a common view of identity across all machines for ACL to be effective