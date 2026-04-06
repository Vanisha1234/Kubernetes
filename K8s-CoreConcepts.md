# K8s-CoreConcepts
A comprehensive guide to **Kubernetes core concepts**, components, objects, and operational commands.  

---

## K8s Architecture

<img width="2100" height="1241" alt="kubernetes_architecture1" src="https://github.com/user-attachments/assets/95ae96a8-c655-4505-ac70-e92c92b9d006" />

---

## ETCD

### Overview
- Distributed, reliable **key-value store**
- Stores cluster state
- No schema, no complex queries
- Fast and highly flexible
### Key Details
| Property | Value |
|--------|------|
| Default Port | `2379` |
| Runs As | Pod or system service |
| Client | `etcdctl` |
| Access | Only via Kube-API Server |
### Role in Kubernetes - stores data regarding the cluster which includes - 
- Nodes
- Pods
- ConfigMaps
- Secrets
- Accounts
- Roles & Bindings
- Cluster metadata
  
> ✅ A change is considered complete **only after it is written to ETCD**
### Operational Commands

To check etcd as pod 
```bash
kubectl get pods -n kube-system
```

---

## KUBE-APISERVER

### Overview
- **Primary management component** in Kubernetes
- Entry point for **all cluster operations**
- All **requests and responses** pass through the Kube-API Server
- **Only component** that communicates directly with **ETCD**
### Responsibilities
- Authenticate users
- Validate API requests
- Retrieve cluster data
- Update ETCD
- Used by **Scheduler** and **Kubelet** to update cluster state
### Operational Commands

Running as a Pod (kubeadm setup)
```bash
kubectl get pods -n kube-system
```
Kube-Api Pod Manifest Location
```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```
Non-kubeadm Setup (Systemd) location
```bash
/etc/systemd/system/kube-apiserver.service
```
To check for running processes
```bash
ps aux | grep kube-apiserver
```

---

## KUBE CONTROLLER MANAGER

### Overview
- A process that **continuously monitors the state** of various components in a Kubernetes cluster
- Works towards bringing the system to the **desired functioning state**
- Multiple controllers are **packaged into a single process** known as the **Kubernetes Controller Manager**
### Example Controllers
#### Node Controller
- Responsible for monitoring the **status of nodes**
- Takes necessary actions to keep applications running
- Receives node status updates via the **Kube-API Server**
- If there is **no response for more than 40 seconds**, the node is marked as **Unreachable**
- If the node remains unreachable for **more than 5 minutes**, the workloads are **replaced** on healthy nodes
#### Replication Controller
- Responsible for monitoring the **status of ReplicaSets**
- Ensures the **desired number of pods** are available at all times within the set
### Controller Packaging
- Various controllers run together inside a **single binary**
- This unified process is called the **Kubernetes Controller Manager**
### Operational Commands

Running as a Pod (kubeadm setup)
```bash
kubectl get pods -n kube-system
```
Pod Manifest Location
```bash
/etc/kubernetes/manifests/kube-controller-manager.yaml
```
Non-kubeadm Setup (Systemd)
```bash
/etc/systemd/system/kube-controller-manager.service
```
Verify Running Process
```bash
ps aux | grep kube-controller-manager
```
---

## KUBE SCHEDULER

### Overview
- Responsible only for deciding which pod goes on which node.
- It does not actually place the pod on a node; it simply selects the best candidate.
- Decides which node has sufficient capacity to accommodate incoming pods.
### Scheduling Logic & Criteria
The scheduler evaluates nodes based on specific logic to ensure optimal placement:
- Resource Prioritization: The scheduler prioritizes the node that will be left with the highest number of available resources even after scheduling the pod.
- Filtering Criteria:
  - Resource Requirements and Limits: Ensures the node meets CPU/Memory requests.
  - Taints and Tolerations: Respects node restrictions.
  - Node Selectors and Affinity: Matches pods to specific node labels or groups.
### Operational Commands

Running as a Pod (kubeadm setup) \
To check the kube-scheduler status when running as a static pod:
```bash
kubectl get pods -n kube-system
```
Manifest and Service Locations \
Depending on your installation method, the configuration is found at: \
Pod Definition File:
```bash
/etc/kubernetes/manifests/kube-scheduler.yaml
```
Non-kubeadm Setup (Systemd):
```bash
/etc/systemd/system/kube-scheduler.service
```
Verify Running Process \
To check for the active scheduler process on the host:
```bash
ps aux | grep kube-scheduler
```

---

## KUBELET

### Overview
- Acts as the sole point of contact between the worker node and the master node.
- The Kubelet on the worker node is responsible for registering the node with the Kubernetes cluster.
- Unlike other components, the Kubelet is always installed manually and not typically run as a container from binaries.
### Responsibilities
The Kubelet manages the lifecycle of pods and maintains communication with the control plane:
- Container Management: Responsible for the loading and unloading of containers.
- Execution Flow:
  - Receives instructions to load a container/pod on a worker node.
  - Requests the Container Runtime (present on the worker node) to pull the necessary image and run the container.
- Monitoring & Reporting:
  - Monitors the health of both nodes and pods.
  - Sends status reports and updates to the Kube-API Server on a timely basis.
### Operational Commands

Since the Kubelet runs as a system service rather than a pod, you can check its status using the following command:
```bash
ps aux | grep kubelet
```

---

## KUBE PROXY

### Overview
- In order to make services accessible throughout the cluster, kube-proxy is used and is installed on every node.
- It is responsible for creating rules on each node every time a new service is created to forward the traffic to that service.
> Kube-proxy manages the network traffic flow by interacting with the node's networking stack:
- It dynamically updates the networking rules to ensure traffic destined for a Service reaches the correct Pod.
- It primarily performs these updates using iptables rules.
### Operational Commands

In most clusters (like those set up with kubeadm), kube-proxy runs as a DaemonSet with a pod on every node. You can verify its status using:
```bash
kubectl get pods -n kube-system
```

---

## PODS

### Overview
- Containers are not deployed directly in Kubernetes; they are encapsulated into Kubernetes objects called Pods.
- A Pod represents a single instance of an application and is the smallest object in a Kubernetes cluster.
- If load increases, a new pod with a container is deployed, or a new node with a new pod is added to the cluster.
  - Scaling up spins up more pods, while scaling down removes them.
- A pod can contain two containers of different types, but generally not of similar types.
- Pods establish all network connectivity automatically when a separate container is created inside them. Even with a single container, pods are used to ensure easy scaling for future architectural changes.
### Basic Commands
To deploy a pod:
```bash
kubectl run nginx --image nginx
```
List Pods:
```bash
kubectl get pods
```
Detailed Info about a Pod:
```bash
kubectl describe pod <pod-name>
```
To delete a Pod:
```bash
kubectl delete pod <pod-name>
```
### Pod Definition File
Kubernetes objects are typically defined using YAML files for declarative management:
```bash
apiVersion: v1 
kind: Pod 
metadata:
  name: myapp
  labels:
      app: myapp-pod
      type: frontend
spec:
  containers:
    - name: nginx-cont
      image: nginx
```
### Managing Pod Configurations
To deploy from a pod definition file:
```bash
kubectl create -f pod.yaml
```
Or
```bash
kubectl apply -f pod.yaml
```

To change a running pod directly:
```bash
kubectl edit pod <pod-name>
```
After saving, apply changes using:
```bash
kubectl apply -f pod.yaml
```

To edit a pod created via vi, vim or nano tool:
```bash
vim pod.yaml
```
Apply changes:
```bash
kubectl apply -f pod.yaml
```

---

## REPLICATION CONTROLLER & REPLICASETS

### Overview
- Helps prevent downtime by maintaining the desired number of pods at all times, ensuring high availability.
- It can also replace an existing unhealthy pod, even if the desired number of pods is just 1.
- It is also responsible for load balancing and scaling across multiple pods, as well as across multiple nodes in case of increasing load.
### Replication-Controller Definition File
```bash
apiVersion: v1 
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
      app: myapp
      type: frontend
spec:
  template: #template block to define the pod definition file for which replications will be created.
     metadata:
       name: myapp
       labels:
          app: myapp-pod
          type: frontend
     spec:
       containers:
         - name: nginx-cont
           image: nginx
  replicas: 3 #number of replicas
```
### Managing Replication Controller Configurations
To create a replication controller definition file:
```bash
kubectl create -f rc.yaml
```
To list replicasets:
```bash
kubectl get replicationcontroller
```
### ReplicaSet Definition File
```bash
apiVersion: apps/v1 
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
      app: myapp
      type: frontend
spec:
  template: #template block to define the pod definition file for which replications will be created.
     metadata:
       name: myapp
       labels:
          app: myapp-pod
          type: frontend
     spec:
       containers:
         - name: nginx-cont
           image: nginx
  replicas: 3 #number of replicas
  selector: #helps replicaset identify what pods falls under it
     matchLabels:
        type: frontend
```
### Managing ReplicaSet Configurations
To update a replicaset:
```bash
kubectl scale --replicas=6 rs.yaml
```
Another way is to update the yaml file itself and run the following command to replace the replicaset:
```bash
kubectl replace -f rs.yaml
```

---

## DEPLOYMENT

### Overview
- Deploying multiple instances of a web server
- Updating multiple Docker instances seamlessly using a rolling update strategy
- Carrying out easy rollbacks in case of bugs and issues
- Making changes to applications running on multiple Docker instances
> Pods help us deploy a single instance of our application; ReplicaSets help us deploy multiple pods; Deployments allow us to upgrade the underlying instances seamlessly.
### Deployment Definition File
```bash
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: myapp-replicaset
  labels:
      app: myapp
      type: frontend
spec:
  template: 
     metadata:
       name: myapp
       labels:
          app: myapp-pod
          type: frontend
     spec:
       containers:
         - name: nginx-cont
           image: nginx
  replicas: 3 #number of replicas
  selector: 
     matchLabels:
        type: frontend
```
### Managing Deployment Configurations
To create a deployment:
```bash
kubectl create -f deployment.yaml
```
To list deployments:
```bash
kubectl get deployments
```
Deployment automatically creates a replicaset hence run the following command to list replicaset created:
```bash
kubectl get replicaset
```
Replicaset automatically creates pods hence run the following command to list the running pods:
```bash
kubectl get pods
```
To list all the created objects at once:
```bash
kubectl get all 
```

---

## SERVICES

### Overview
- Enables communication between various components within and outside the application, such as connecting frontend pods with backend pods.
- Helps connect different applications and users.
- Services enable loose coupling between microservices in our application.

> In case of external communication
- If we need to communicate with an application hosted on a pod from outside the Kubernetes cluster, we can use Services instead of logging into the node and accessing it directly.
### NodePort Service
Since the use cases of Services include listening on a port of the node and forwarding requests to the port on the pod hosting the application, this type of Service is known as a NodePort Service.
- TargetPort – The port on the pod where the Service forwards the request.
- Port – The port on the Service itself; the Service has a virtual IP assigned to it known as ClusterIP.
- NodePort – The port on the node. NodePort has a range of 30000–32767.
### Service Definition File of type NodePort
```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort #type of service
  ports:
    - targetPort: 80 #if targetPort is not provided, it will automatically pick up the same port of the port.
      port: 80 #this is important to provide
      nodePort: 30008 #if not provide, will be automatically set to any port between the valid range
  selector: #labels and selectors will be used to link the service to the pod to which the request is to be forwarded
      app: myapp
      type: frontend
```
### Managing Service Configurations
To create a service:
```bash
kubectl create -f svc.yml
```
To list the services:
```bash
kubectl get services
```
To describe a service:
```bash
kubectl describe service serviceName
```
We can now browse the application using the IP of the Node and the Port on the Node.
### ClusterIp Service
- A virtual IP is created inside the cluster and is used to enable communication between different services, such as a set of frontend servers communicating with a set of backend services.
- In cases where multiple services are running—for example, a set of frontend web servers and a set of backend web servers—both need to communicate with each other. However, pod IPs are not static and may change if a pod is replaced, making them unreliable for internal communication.
- Secondly, it becomes difficult to decide which pod instance to forward traffic to when all instances are the same. Hence, Cluster IP service helps in grouping similar pods together and provides a single interface for communication.
- Each Service is assigned a name and an IP, which will be used by pods for communication.
### Service Definition File of type ClusterIp
```bash
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP #if type of service is not specified;this is the default svc type
  ports:
    - targetPort: 80 #port where backend is exposed
      port: 80 
  selector: #to link svc to backend pods
      app: myapp
      type: backend
```
The svc can be accessed by the pods using the clusterIP or the service name.
### Load Balancer Service
- Helps distribute load across various web servers.
- In case of multiple pod instances running the same application, each pod has the same labels, which are used in the LoadBalancer Service to link all the pods.
- Once the pods are linked using labels, the Service automatically uses all the pods as endpoints to forward requests.
- This Service acts as a built-in load balancer to distribute load.
- In case pods are present on different nodes inside the cluster, no additional configuration is required since Kubernetes automatically spans across the nodes in the cluster and maps the target port to the same NodePort on all nodes. This way, the application can be accessed through any node IP using the same port.
- Services are automatically updated if pods are added or removed.

---

## NAMESPACES

### Overview
- An isolated environment that contains all the resources like pods, services (svcs), and deployments.
- There can be various namespaces like dev, prod, and test.
- Namespaces are created to isolate these environments so that users don’t accidentally make changes to another environment.
- The resources within a namespace can refer to each other by their names.
- To connect to a Service in another namespace (e.g., a web pod connecting to a DB pod in another namespace), we need to use:
```bash
servicename.namespace.svc.cluster.local
```
### Namespace Definition File
```bash
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
### Managing Namespace Configurations
To list the pods in a specific namespace:
```bash
kubectl get pods --namespace=kube-system
```
To create a pod in a specific namespace 
```bash
kubectl create -f pod.yml --namespace=blue-namespace
```
> To avoid defining namespaces in command line repeatedly, define the namespace under the metadata section of the pod definition file.
To create using definition file:
```bash
kubectl create -f ns.yml
```
To create a namespace directly without creating def. file
```bash
kubectl create namespace <Namespace Name>
```
To avoid specifying ns repeatedly while running commands use:
```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```
To view pod in all the namespaces:
```bash
kubectl get pods --all-namespaces
```
### Resource Quota
To limit the resources of a namespace, such as CPU or memory usage, create a ResourceQuota.
### ResourceQuota Definition File
```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10 Gi
```   
To create resource quota via definition file:
```bash
kubectl create -f rq.yaml
```

---
  
## IMPERATIVE & DECLARATIVE
Imperative Approach means specifying what and how to do.
Declarative Approach means specifying what to do instead of specifying the details; the rest is handled by the system itself.

### Example of Imperative Approach
- Imperative commands run once and require long commands in case of complex environments
- They are limited to the user who ran these commands and are difficult to keep track of \
> To create an object
Pod creation:
```bash
kubectl run --image=nginx nginx
```
Deployment creation:
```bash
kubectl create deployment --image=nginx nginx
```
Service creation to expose deployment:
```bash
To create a service to expose deployment - kubectl expose deployment nginx --port 80
```
> To update an object
Edit the existing object:
```bash
kubectl edit deployment nginx
```                                 
Scaling a deployment:
```bash
kubectl scale deployment nginx --replicas=5
```                          
Updating an image on deployment:
```bash
kubectl set image deployment nginx nginx=nginx:1.18
```
> Using files
To create an object specifying file:
```bash
kubectl create -f nginx.yaml
```
To edit an object specifying file:
```bash
kubectl replace -f nginx.yaml
```
To delete an object specifying file:
```bash
kubectl delete -f nginx.yaml
```
To create , update and delete an existing object:
```bash
kubectl apply -f nginx.yaml
```
We create YAML files, which are easy to keep track of, share, and update.
> Ways of Editing an Existing Object
1. Using ```kubectl edit deployment nginx```
- This command opens a YAML file similar to the one used to create the object, but with additional fields
- This is not the original file; it is stored in Kubernetes memory
- Changes made here are applied live \
Difference:
- Changes made in Kubernetes memory are not recorded in the local file
- The local YAML file remains unchanged  
2. Better approach:
- Make changes in the local YAML file
- Run: ```kubectl replace -f nginx.yaml```
3. To completely delete of recreate an object,
- Run: ```kubectl replace --force -f nginx.yaml```

### Examples of Declarative Approach
- In the declarative approach, we use the same YAML files, but instead of ```create``` or ```replace```, we use:
```kubectl apply -f nginx.yaml```
- ```Kubectl apply``` is intelligent enough to create the object if it does not exist
- To create multiple objects:
  - Specify them in a directory
  - Run: ```kubectl apply -f /path/to/directory```
- To update the existing file:
  - Make changes in the config file
  - Run: ```kubectl apply -f nginx.yaml```

### Useful Commands
To create a pod:
```bash
kubectl run nginx --image=nginx
```
Generate POD manifest YAML (without creating it):
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```
To create a deployment:
```bash
kubectl create deployment --image=nginx nginx
```
Generate Deployment YAML (without creating it):
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```
To save generated YAML to a file:
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```
Generate deployment with 4 replicas:
```bash
kubectl create deployment nginx --image=nginx --replicas=4
```
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379:
```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```
Create a Service named nginx of type NodePort to expose pod nginx’s port 80 on port 30080 on the nodes:
```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
To create a service and simultaneously create a pod and expose it: (Service name and pod name will be the same)
```bash
kubectl run httpd --image=httpd:alpine --port=80 --expose
```

---

## KUBECTL COMMAND
- To list all resources – ```kubectl api-resources```; lists names, short names, API versions, and other details about all the resources
- To explain an object – ```kubectl explain pod```; for detailed information – ```kubectl explain pod --recursive```

### KUBECTL APPLY COMMAND
- ```kubectl apply``` stores the changes made to the last applied configuration file.
- All three files - the local file, the last applied configuration file, and the Kubernetes memory file are compared, and the final changes are identified and applied
The last applied configuration file helps keep track of what changes are made when the files are compared.

---
