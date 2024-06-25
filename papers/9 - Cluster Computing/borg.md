# Borg, Omega, and Kubernetes (2016) 
Lessions learned from container-management system 
* Borg: built to manage long-running services and batch jobs
* Omega: improve the software engineering of Borg ecosystem
* K8s: open-source, tool for developers

## Containerization 
* Runtime isolation (`cgroup`) and image (i.e. data)
* Transfrom DC from machine-oriented to app-oriented
    *  encapsulate app environment, abstract OS and machine 
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
    *  Don't number containers, give them labels
    *  Be careful with ownership (i.e. task and job)
    *  Don't expose raw state
        *  Borg: monolithic component knows every API op (i.e. Borg master) 
        *  Omega: no centralized component except store
            *  shared state
            *  but all logic and semantics pushed to client, each component R/W independently 
        *  K8s: middle ground
            *  centralized API server
            *  but control logic distributed among multi-scheduler   
