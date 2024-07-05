# Borg, Omega, and Kubernetes (2016) 

Link: https://dl.acm.org/doi/pdf/10.1145/2898442.2898444

June 28th, 2024

Lessions learned from container-management system 
* Borg: built to manage long-running services and batch jobs
  * Even so, the Borg container image is not quite as
  airtight as it could have been: applications share a so-called base image that is installed once on the machine rather than being packaged in each container. This base image contains utilities such as tar and the libc library, so upgrades to the base image can affect running applications and have occasionally been a significant source of trouble.
* Omega: improve the software engineering of Borg ecosystem
  * Omega stored the state of the cluster in a centralized Paxos-based transaction-oriented store that was accessed by the different parts of the cluster control plane (such as schedulers), using optimistic concurrency control to handle the occasional conflicts
  * Decoupling allows Borg's functionality to be broken into peers, rather than funneling every change through a centralized Borg master.
* K8s: open-source, tool for developers
  * Like Omega, Kubernetes has at its core a shared persistent store, with components watching for changes to relevant objects. In contrast to Omega, which exposes the store directly to trusted control-plane components, state in **Kubernetes is accessed exclusively through a domain-specific REST API** that applies higher-level versioning, validation, semantics, and policy, in support of a more diverse array of clients.
  * Focuses on the experience of users.
  * A modern container is more than just an isolation mechanism: it also includes an image—the files that make up the application that runs inside the container. Within Google, MPM (Midas Package Manager) is used to build and deploy container images
  * **runtime isolation** and **image**
  *  Compare this to having to ssh into a machine to run top. Though it is possible for developers to ssh into their containers, they rarely need to.

## Containerization 
* Runtime isolation (`cgroup`) and image (i.e. data)
* Transfrom DC from machine-oriented to app-oriented
    *  container as a unit of management
        *  flexibility on OS / HW updates
        *  better application monitoring
        *  load balancing, scheduling (i.e. `pods`)
    *  orchestration (k8s) 
        *  unified API - metadata, status, spec per object
        *  composable building block - pods, replica controller, services 
        *  consistency: reconciliation loop
            *  small control loop to check system state (compare desired v.s observe)   
        *  opt for decentralized control 
            *  adaptable and robust
            *  e.x. pod autoscaler: decide pod count, replication controller: perform replication
            *  v.s. centralized Borgmaster 
*  Things to avoid
    *  Don't make container system manage port numbers
       *  Learning from our experiences with Borg, we decided that Kubernetes would allocate an IP address per pod, thus aligning network identity (IP address) with application identity.
    *  Don't number containers, give them labels
    *  Be careful with ownership (i.e. task and job)
       *  In Kubernetes, pod-lifecycle management components such as replication controllers determine which pods they are responsible for using label selectors, so multiple controllers might think they have jurisdiction over a single pod
    *  Don't expose raw state
        *  Borg: monolithic component knows every API op (i.e. Borg master) 
        *  Omega: no centralized component except store
            *  shared state
            *  but all logic and semantics pushed to client, each component R/W independently 
        *  K8s: middle ground
            *  centralized API server
            *  but control logic distributed among multi-scheduler 
   *  Open Hard problems:
      *  Managing configuraitons: the set of values supplied to applications, rather than hard-coded into them.  
         *  We believe the most effective approach is to accept this need, embrace the inevitability of programmatic configuration, and maintain a clean separation between computation and data.
      *  Hard to keep dependency information up-to-date. 
