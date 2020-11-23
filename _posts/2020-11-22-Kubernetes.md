---
layout: post
category: Kubernetes
tag: Kubernetes, k8, K8
title: Kubernetes
---
Kubernetes is an open source **container orchestration tool** which was originally developed by **Google**. It manages 
containers (docker or from other technologies) or **containerized applications** in different **deployment environments** 
such as *physical machines*, *virtual machines*, *cloud environments* or even hybrid deployment environment.

##### What problems does Kubernetes solve?
The rise of microservices cause increased usage of container technologies.
The containers offer the perfect host for the small, independent applications like microservices.
The rise of containers and microservice technoliogies resulted in applications that now comprises of hundreds and thousands of containers.
Managing such huge containers across multiple environments using scripts and self made tools can be really complex and sometimes impossible.
Such scenario caused the need for having containers orchestration technologies.

##### What are the tasks of an orchestration tool?
- **High Availability** or no downtime:
    The application has no downtime and is always accessible by the users.
- **Scalability** or high performance:
    The application has high performance. It loads fast and user has a high response rates from the application. 
- **Disaster recovery** - backup and restore:
    If an infrastructure has some problem, then it needs to have some mechanism to pick up the data and restore it to latest state so that the application does not loose any data.
    The contanirized application can run from the latest state after the recovery.

##### Kubernetes Components
###### Node and Pod
A **Pod** is the smallest unit of K8s. It is an abstraction over a container. It creates a running environment or a layer on top of the container. Usually the norm is to create an application per Pod.
Kubernetes offers out of the box virtual network, which means each Pod gets its own IP address (not the container). Each Pod can communicate with each other using the IP address which is an internal IP address.
A container can crash easily due to various reason and a new one gets created in its place with a new IP address.
This is inconvininent if one is communicating using the IP address as one has to adjust it everytime when the Pod restarts.
Therefore, another component of Kubernetes called **Service** is used.

A **Service** is a static/permanent IP address that can be attached to each Pod.
The lifecycle of Pod and Service are not connected. Even if the Pod dies, the Service and its IP address would stay. So there is no need to change the endpoint anymore.
A Service can be either **External** or **Internal**.

An *External Service* is the service that opens the communication from external sources.
For services that should not be open to public request, an *Internal Service* is created.

###### Ingress
Ingress is another component in Kubernetes. Instead of a Service, the external request goes first to the **Ingress** and it does the forwarding to the Service.

###### ConfigMap
It is an external configuration of the application. A **ConfigMap** usually contain configuration data like URLs of the database or some other services that an application uses.

###### Secret
Part of the external configuration could contain sensitive data such as database username and password. 
Putting credentials into ConfigMap in a plain-text format would be insecure.
For this purpose, Kubernetes has another component called **Secret**.
It is used to store secret data and it is stored in base64 encoded format.

###### Data storage (Volumes)
It attaches a physical storage on a hard drive to the Pod. This storage can be either on the local machine or on a remote storage, outside of the K8s cluster.  
Note that Kubernets does not manage any data persistence.

###### Deployment and Stateful Set
In case of downtime where a user can't reach the application.
This is where the distributed systems and containers comes handy where everything is replicated in multiple servers instead of having a single point of failure.
The replica is connected to the same Service. Recall a Service is like a persistant static IP address with the DNS name.
A Service is also a load balancer.

In order to create the second replica of the application, we wouldn't create a second Pod. But instead we would define a bluprints for the application Pod and specify how many replicas of that Pod you'd like to run.
That blueprint is called **Deployment**.

In practice, you would not be working with Pods, you will be creating Deployments where we can scale up or scale down the number of replicas of Pod that one need.
It is an abstraction of Pods.

Note that a database cannot be replicated via Deployment because the database has a State, which is its data.
Meaning that, if we have clones of database, they will all need to access the same shared data storage. And there we would need some mechanism that manages which Pods are reading and writing to/from the storage in order to avoid data inconsistencies.
That mechanism in addition to replicating feature is offered by another Kubernetes component called **Stateful Set**.
This component is meant specifically for applications like databases (such as MongoDb, MySQL, ElasticSearch, etc.).

Therefore, **Deployment** for state**LESS** Apps and **StatefulSet** for state**FUL** Apps or Databases.
However, deploying StatefulSet is not easy and is tedious. DB are often hosted outside of K8s cluster.

### Worker Node Processes
Each Node has multiple applications or Pods on it.
3 processes must be installed on every Node that are used to schedule and manage those Pods.
Nodes are the cluster services that actully do the work therefore sometimes  a.k.a. Worke Nodes.
#### 1. Container Runtime
Application pods have containers running inside, a container runtime needs to be installed on every node.

#### 1. Kublet
The process that actually schedule the Pods underneath is Kublet.
It interacts with both the container and node
Kublet starts the pod with a container inside. 
It assigns resources from the node to the container like CPU, RAM and storage
    
#### 3. Kube Proxy
It Forwards the requests intelligently forwards to make sure the communication also works in a performant way with low overhead.

### Master Node Processes
4 processes run on every master node.
#### 1. API Server
Client interacts to the API server. It is the cluster gateway and acts as a gatekeeper for authentication.
API server validates the request and then forwards the request to other processes to schedule the Pod.
This is the only entry point to the cluster.

#### 2. Scheduler
After API Server validates the request, the request is handed over to the Scheduler in order to start the application Pod on one of the Nodes.
Instead of randomly assigning to any node, Scheduler intelligently decides on which specific Worker Node the new Pod or component should be scheduled.
The process that actually does the scheduling / starts the Pod with the container is the Kubelet.

#### 3. Controller Manager
Detect cluster state changes (like crashing of Pods) and tries to recover cluster state as soon as possible.
It makes the request to the Scheduler to reschedule those dead Pods. 

#### 4. etcd
It is the Key-Value store of the cluster state.
etcd is the **cluster brain**. 
Cluster changes get stored in the key value store.

**Note**: *The actual application data is not stored in etcd.*

### Example Cluster Set-Up
 - 2 Master Nodes
 - 3 Worker Nodes
 
##### Adding new Master/Node server:
1) Get new bare server
2) Install all the master/worker node processes
3) Add it to the cluster

#### References
- https://www.youtube.com/watch?v=X48VuDVv0do&ab_channel=TechWorldwithNana
