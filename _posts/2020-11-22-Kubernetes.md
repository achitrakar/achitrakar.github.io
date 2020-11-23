### Introduction to Kubernetes
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


### Minikube and kubectl - Local Setup
#### minikube
Minikube runs both Master and Worker processes.
- Creates Virtual box on your laptop
- Node runs in that Virtual box
- 1 Node K8s cluster
- For testing purpose

#### kubectl
One of the Master Processes called API Server is the entry point to the K8 cluster.
Way to talk to API Server is through different clients such as UI, Kubernetes API or CLI (kubectl).
kubectl is the most powerful of 3 clients.

#### Install Minikube
Installation Guide links: 
- https://kubernetes.io/docs/tasks/tools/install-minikube/
- https://kubernetes.io/docs/tasks/tools/install-kubectl/

**Note**: A **Virtulization** on your machine is needed!
```bash
$ brew update
$ brew install hyperkit
$ brew install minikube
```

```bash
$ minikube start --vm-driver=hyperkit
* minikube v1.15.1 on Darwin 10.15.7
* Using the hyperkit driver based on existing profile
* Starting control plane node minikube in cluster minikube
* Updating the running hyperkit "minikube" VM ...
* Preparing Kubernetes v1.19.4 on Docker 19.03.13 ...
* Verifying Kubernetes components...
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
$
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   16m   v1.19.4
$ 
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
$
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-12T01:09:16Z", GoVersion:"go1.15.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:09:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"
$
```
- minicube CLI: for start up.deleting the cluster
- kubectl CLI: For configuring the minikube cluster

###### Creating  Deployment and Pod
```
$ kubectl create deployment nginix-depl --image=nginx
```
- **Blueprint** for creating Pods
- **Most basic configuration** for deployment (name and image to use)
- **Rest defaults**

###### Check Deployment and Pod
```
$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           32s
$
kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-5c8bf76b5b-2kp2m   1/1     Running   0          56s
```

###### Check Replicaset
```
$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   1         1         1       8m32s
``` 
- Replicaset is managing the replicas of a Pod

##### Layers of Abstraction
- **Deployment** manages a ReplicaSet
- **ReplicaSet** manages all the replicas of that Pod
- **Pod** is an abstraction of a **Container**
- Everything **below** Deployment is handled by Kubernetes 

For example, the image that it uses will have to edit in the Deployment directly and not in the Pod.
We will get an auto-generated configuration file of the deployment with default values. Recall when we created a deployment via command line, we just gave two options; everything else is default and auto-generated by the Kubernetes.
```
$ kubectl edit deployment nginx-depl
deployment.apps/nginx-depl edited
$
$ kubectl get pod
NAME                          READY   STATUS              RESTARTS   AGE
nginx-depl-5c8bf76b5b-2kp2m   1/1     Running             0          19m
nginx-depl-7fc44fc5d4-clj6m   0/1     ContainerCreating   0          8s
$
$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-7fc44fc5d4-clj6m   1/1     Running   0          79s
$
$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   0         0         0       21m
nginx-depl-7fc44fc5d4   1         1         1       2m
``` 

##### Debugging Pods
*kubectl logs [pod name]* 
```
$ kubectl logs nginx-depl-7fc44fc5d4-clj6m
$
$ kubectl create deployment mongo-depl --image=mongo
deployment.apps/mongo-depl created
$
$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
mongo-depl-5fd6b7d4b4-42g2b   1/1     Running   0          26s
nginx-depl-7fc44fc5d4-clj6m   1/1     Running   0          7m18s
```
*kubectl logs [pod-name]*
```
$ kubectl logs mongo-depl-5fd6b7d4b4-42g2b
{"t":{"$date":"2020-11-22T20:02:24.815+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2020-11-22T20:02:24.819+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
...
{"t":{"$date":"2020-11-22T20:02:25.388+00:00"},"s":"I",  "c":"INDEX",    "id":20345,   "ctx":"LogicalSessionCacheRefresh","msg":"Index build: done building","attr":{"buildUUID":null,"namespace":"config.system.sessions","index":"_id_","commitTimestamp":{"$timestamp":{"t":0,"i":0}}}}
{"t":{"$date":"2020-11-22T20:02:25.388+00:00"},"s":"I",  "c":"INDEX",    "id":20345,   "ctx":"LogicalSessionCacheRefresh","msg":"Index build: done building","attr":{"buildUUID":null,"namespace":"config.system.sessions","index":"lsidTTLIndex","commitTimestamp":{"$timestamp":{"t":0,"i":0}}}}
```
*kubectl describe pod [pod-name]*
```
$ kubectl describe pod mongo-depl-5fd6b7d4b4-42g2b
Name:         mongo-depl-5fd6b7d4b4-42g2b
Namespace:    default
Priority:     0
Node:         minikube/192.168.64.2
Start Time:   Sun, 22 Nov 2020 14:02:08 -0600
Labels:       app=mongo-depl
              pod-template-hash=5fd6b7d4b4
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:           172.17.0.3
Controlled By:  ReplicaSet/mongo-depl-5fd6b7d4b4
Containers:
  mongo:
    Container ID:   docker://6332284f8d5aaa61714711dcd5f66f2efdf3f51757955fb32875b3dd22e05a09
    Image:          mongo
    Image ID:       docker-pullable://mongo@sha256:7aa0d854df0e958f26e11e83d875d0cccc53fab1ae8da539070adfc41ab58ace
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 22 Nov 2020 14:02:24 -0600
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fvksv (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-fvksv:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fvksv
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m51s  default-scheduler  Successfully assigned default/mongo-depl-5fd6b7d4b4-42g2b to minikube
  Normal  Pulling    2m51s  kubelet            Pulling image "mongo"
  Normal  Pulled     2m35s  kubelet            Successfully pulled image "mongo" in 15.409045176s
  Normal  Created    2m35s  kubelet            Created container mongo
  Normal  Started    2m35s  kubelet            Started container mongo
```
Use interactive terminal: `kubectl exec -it [pod-name] -- bin/bash` to get into the container as a root user.
```
$ kubectl exec -it mongo-depl-5fd6b7d4b4-42g2b -- bin/bash
root@mongo-depl-5fd6b7d4b4-42g2b:/# 
```
*kubectl delete deployment [name]*
```
$ kubectl delete deployment mongo-depl
deployment.apps "mongo-depl" deleted
$
$ kubectl get depeloyment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           38m
$
$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-7fc44fc5d4-clj6m   1/1     Running   0          20m
$
$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   0         0         0       39m
nginx-depl-7fc44fc5d4   1         1         1       20m
```

##### Creating deployment using a configuration file  
Usage: *kubectl apply -f [file name]*
```
$ cat nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
$ 
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
$
$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-644599b9c9-9t4sl   1/1     Running   0          45s
$
```
You can edit the existing deployment file and try to apply it again after the update. For example change the `replicas` to be 2 instead.
```
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment configured
$
```
**Note** that K8 knows when to create or update deployment.
```
$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           5m9s
```
It is the old deployment.
```
$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-644599b9c9-9t4sl   1/1     Running   0          5m44s
nginx-deployment-644599b9c9-gsfcz   1/1     Running   0          3m7s
```
Old one is still there and the new one got created as replica count was increased.

#### Summarize kubectl commands
- CRUD commands
    - **Create** deployment              - `kubectl create deployment [name]`
    - **Edit** deployment                - `kubectl edit deployment [name]`
    - **Delete** deployment              - `kubectl delete deployment [name]`
- Status of different K8s components
    - `kubectl get nodes | pod | services | replicaset | deployment`
- Debugging pods
    - **Log** to console                 - `kubectl logs [pod name]`
    - Get **Interactive Terminal**       - `kubectl exec -it [pod name] -- bin/bash`
    - Get **info** about pod             - `kubectl describe pod [pod-name]`
- Use configuration file for CRUD
    - **Apply** a configuration file     - `kubectl apply -f [file name]`
    - **Delete** with configuration file - `kubectl delete -f [file name]`

### YAML Configuration File in Kubernetes
#### The three parts of configuration file
- nginx-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels: ...
spec:
  replicas: 2
  selector: ...
  template: ...
```
- ngnix-service.yaml
```
apiVersion: apps/v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector: ...
  ports: ...
```

#### Conecting Deployments to Service to Pods

### Summary
   - What is Kubernetes?
        - The need for a container orchestration tool
            - Trend from **Monolith** to **Microservices**
            - Increased usage of **containers**
   - K8s Architecture
   - Main K8s Components
        - Main Kubernetes Components summarized
            - Volumes
            - Pod
            - Service
            - Ingress
            - ConfigMap
            - Secrets
            - Deployment
            - StatefulSet
   - Minikube and kubectl - Local Setup
        - Minikube
            - Creates Virtual box on your laptop
            - Node runs in that Virtual box
            - 1 Node K8s cluster
            - For testing purpose
   - Main Kubectl Commands - K8s CLI
        -  **Create** and **debug** Pods in a minikube cluster
        - CRUD commands
            - **Create** deployment - *kubectl create deployment [name]*
            - **Edit** deployment   - *kubectl edit deployment [name]*
            - **Delete** deployment - *kubectl delete deployment [name]*
        - Status of different K8s components
            - *kubectl get nodes | pod | services | replicaset | deployment*
        - Debugging pods
            - **Log** to console           - *kubectl logs [pod name]*
            - Get **Interactive Terminal** - *kubectl exec -it [pod name] -- bin/bash*
   - K8s YAML Configuration File
   - Hands-On Demo
        - Browser Request Flow through the K8s components


### Advanced Concepts
   - K8s Namespaces - Organize your components
        - Resources grouped in Namespaces
            - K8 cluster:
                - Database
                - Monitoring
                - Elastic Stack
                - Nginx-Ingress
   - K8s Ingress
        - Multiple paths for same host
            - http://<myapp.com>/analytics
                - Analytics service ==> Analytics pod
            - http://<myapp.com>/analytics
                - Shopping service ==> Shopping pod    
   - Helm - Package Manager
        - Kubernetes Cluster:
            - my-app pod
            - my-app service
   - Volumes - Persisting Data in k8s
   - K8s StatefulSet - Deploying Stateful Apps
   - Kubernetes Services
        - What is a Kubernete Servie and when we need it?
        - Different Service types explained:
            - **ClusterIP** Services
            - **Headless** Services
            - **NodePort** Services
            - **LoadBalancer** Services
        - Difference between them and when to use which one
        
#### References
- https://www.youtube.com/watch?v=X48VuDVv0do&ab_channel=TechWorldwithNana
