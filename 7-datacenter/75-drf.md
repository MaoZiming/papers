# Dominant Resource Fairness: Fair Allocation of Multiple Resource Types (2011) 

Link: https://amplab.cs.berkeley.edu/wp-content/uploads/2011/06/Dominant-Resource-Fairness-Fair-Allocation-of-Multiple-Resource-Types.pdf

Read: July 10th, 202* 

* To address the problem of fair resource allocation in a system containing different resource types, the paper proposes Dominant Resource Fairness (DRF), a generalization of max-min fairness to multiple resource types. 
* All the prior works on fair resource allocation focuses on a single resource. 
* For systems with multiple resources, prior works (e.g. Hadoop and Dryad) use a single resource abstraction: allocate resources at the level of fixed-size partitions of the nodes called slots. 
 
DRF **allocates resources according to agents’ proportional demands, in a way that equalizes the shares that agents receive of their most highly demanded resources**. 

* Dominant resource: resource user has the biggest share of
* Dominant share: fraction of the dominant resource user is allocated
* **DRF approach:** equalize the dominant share of users
  * or maximize the minimum dominant share across all users. 
* **Algorithm:** whenever there are available resources, schedule a task to the user with the smallest dominant share

### Properties 
* **Sharing incentive**: user is no worse off than a cluster with $\frac{1}{n}$ resources
* **Strategy proof**: user should not benefit by lying about demands
* **Pareto efficiency**: not possible to increase one user without decreasing another
* **Envy free**: user should not desire the allocation of another user

### Examples

* System resources = <9, 18>
* Task A requires <1, 4>, and Task B requires <3, 1>
* DRF repeatedly selects the user with the lowest dominant share to launch a task, until no more tasks can be allocated

### Comparison and extension

* In section 5, DRF is compared to assert fairness (equalize the aggregate resources allocated to each user) and CEEI (Competitive Equilibrium from equal incomes)
  * Assert fairness: equal shares of different resources are worth the same, i.e., that 1% of all CPUs worth is the same as 1% of memory and 1% of I/O bandwidth. Asset Fairness then tries to equalize the aggregate resource value allocated to each user. 
    * Users budget are user's demand vector * relative capacity of the resources. 
    * Violate sharing incentive. 
      * The user might be better off with a static partitioning of the cluster. 
  * CEEI: picks feasible allocations that maximize the product of users' utility.
    * not strategy-proof. 
    * User can lie about their demand vectors. 
* Weighted DRF: a user's dominant share is modified with a weight. 

### Connections 
DRF can be applied to general resource allocation in multi-tenant setup to ensure fair usage of multiple resource types. For example, in a cloud environment where VMs are competing for CPU, memory, and bandwidth, DRF can allocate resources in such a way that no single VM or container can monopolize any single resource, thus providing a balanced and fair environment.

### Thoughts:

* Furthermore, any policy that satisfies the sharing incentive property also provides performance isolation, as it guarantees a minimum allocation to each user (i.e., a user cannot do worse than owning n1 of the cluster) irrespective of the demands of the other users.

* Performance isolation remains on theory. 