# Dominant Resource Fairness: Fair Allocation of Multiple Resource Types (2011) 
To address the problem of fair resource allocation in a system containing different resource types, the paper proposes Dominant Resource Fairness (DRF), a generalization of max-min fairness to multiple resource types. 

DRF **allocates resources according to agents’ proportional demands, in a way that equalizes the shares that agents receive of their most highly demanded resources**. 

- Dominant resource: resource user has the biggest share of
- Dominant share: fraction of the dominant resource user is allocated
- **DRF approach:** equalize the dominant share of users
- **Algorithm:** whenever there are available resources, schedule a task to the user with the smallest dominant share

<img width="473" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/8d831a98-01a2-4239-b674-7d8c48ff637c">


### Properties 
- **Sharing incentive**: user is no worse off than a cluster with 1 / n resources
- **Strategy proof**: user should not benefit by lying about demands
- **Pareto efficiency**: not possible to increase one user without decreasing another
- **Envy free**: user should not desire the allocation of another user

### Examples
<img width="449" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/58ebc99d-051e-434a-a40a-51e2c55890b3">



- System resources = <9, 18>
- Task A requires <1, 4>, and Task B requires <3, 1>
- DRF repeatedly selects the user with the lowest dominant share to launch a task, until no more tasks can be allocated

### Connections 
DRF can be applied to general resource allocation in multi-tenant setup to ensure fair usage of multiple resource types. For example, in a cloud environment where VMs are competing for CPU, memory, and bandwidth, DRF can allocate resources in such a way that no single VM or container can monopolize any single resource, thus providing a balanced and fair environment.
