# kubernetes
Deploy and manage applications in the form of containers.

Kubernetes is an open source container orchestration system for automating appliccation deployment, scalling and management.
It was desgined by google and is now maintained by cloud native computing foundation.

The following things are required for Kubernetes for developers - configMaps, secrets, ServiceAccounts, Multi container pods, readiness and liveness pods, logging & monitoring jobs, services and networking.

### What is Node: 
It is a machine (physical/ virtual) which kubernetes is installed. Node is worker machine and that containers where launched by kubernetes.

### What is Cluster: 
Set of / group of nodes for scalling and sharing the load.

### What is Master: 
It is responsible for managing clusters. Master is another node were kubernetes installed and configured as a master.
It is responsible for actual orchestration of containers on workers nodes.

### What is Kubernetes Components: 
         API server - Act as frontend for kuberneters
         etcd service - Destributed Key & value store to manage clusters
         Kubelet - Agent to run on each nodes
         Container Runtime - To run the containers (docker) monitoring
         Controllers - Responsible for orchistration, scalling, endpoint
         Schedulers - Destributing work/ containers across multiple nodes


### Master Vs worker Nodes
 * Worker nodes -> where containers are hosted
 * Master is where Kube-API server installed
 * Master contains -> API server, etcd, controller, schedulers
 * Worker nodes contains -> Kublet, container runtime(Docker)
 * Kubectl: It is commandline tool to deploy and manage applications.
Example: 
         kubectl run hello-world
         kubectl cluster-info
         
### What is POD
The containers are encapsulated into kubernet object called POD.
POD: Single instance of application OR smallest object in kubernetes.
It is recommeded to host one application in one POD. Sometimes we can host multi applications in same POD.
