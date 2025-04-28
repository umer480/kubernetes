# POD 

## What is POD:
A **pod** is the smallest and simplest Kubernetes object that you create and manage in kubernetes.
**POD** is nothing but a logical boundary for containers. - POD contains multiple containers - You can say POD is a collection of containers.


# POD Very Basic Diagram:

![image](https://github.com/user-attachments/assets/ab37b00e-5762-45be-afff-5e1a6bbb6dce)


**A pod makes sure those containers:**
-Share the same network (means - same IP address, use the same network  interface)
-Share storage if needed
-are scheduled together (they run on the same machine)


You usually put **one container** in a pod. But sometimes, you might put multiple containers that must work closely together.

+-------------------------------------------------+
|                     Pod                         |
|                                                 |
|  +-----------------+   +---------------------+  |
|  |  Container A    |   |    Container B      |  |
|  |  (e.g., App)    |   |  (e.g., Sidecar)    |  |
|  +-----------------+   +---------------------+  |
|                                                 |
|  Shared Network (IP) and Storage Volumes        |
+-------------------------------------------------+

**In this diagram:**

**Container A** might be your main application.
**Container B** could be a helper process, like a logging agent.

**NOTE:** Both containers share the same network and storage resources, facilitating seamless communication and data sharing



## Get the API Version and Kind for any Kubernetes object:
```bash
kubectl explain pods
```

![image](https://github.com/user-attachments/assets/9c020a01-cc4e-4771-be11-3e2ef4286d0b)


## POD Creation
1- imperative way
2- Declarative way

## Create POD - Imperative way
```bash
kubectl run mypod --image-nginx
```
# watch command output:
```bash
kubectl get pods --watch 
```

## Create POD - Declarative way ( + Print out ready made YAML)
```bash
kubectl run podname --image=nginx --dry-run=client -o yaml > mypod.yml
```





![image](https://github.com/user-attachments/assets/ae3cd716-ed2e-4c60-abf1-094103523777)





## Communication within same pod / communication between multiple containers s within same POD:

## Shared Network Namespace
One of the key features that enable container communication within a Pod is the shared network namespace. This means that all containers within a Pod share the same network resources, such as IP address, network interfaces, and ports. This shared environment allows containers to communicate with each other directly using localhost or 127.0.0.1 as if they were running within the same operating system instance.

**Loopback Interface**: Containers within a Pod can communicate via the loopback interface, using the localhost address. For example, if one container is running a service on port 8080, another container within the same Pod can access this service by connecting to localhost:8080.

**Shared IP Address:** Every Pod is assigned a unique IP address within the Kubernetes cluster, and all containers in the Pod use this IP. This eliminates the need for complex networking configurations between containers in the same Pod.


**Example : communication via localhost of web and db server**
consider a scenario where one container in a Pod runs a **web server on port 8080**, and another container runs a database server on port 3306. The web server can connect to the database using **localhost:3306**, leveraging the shared network namespace.



# Communication across different PODs
Each Pod gets its own IP address, allowing communication with other Pods in the cluster.


## Pods are ephemeral:
pods are ephemeral by nature, if a pod (or the node it executes on) fails, Kubernetes can automatically create a new replica of that pod to continue operations.



## Basic Pod Configuration Example:
```bash


apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
spec:
  containers:
  - name: app-container
    image: my-app
    ports:
    - containerPort: 8080
  - name: helper-container
    image: busybox
    command: ['sh', '-c', 'echo Hello from helper']

```


## POD Health States:

![pod-animation](https://github.com/user-attachments/assets/35c5d3ae-7671-4702-bb0f-71baa2a8870c)





## PODs - Commands

kubectl get pods
kubectl get pods -A

# Access a Container Within a Pod:
kubectl exec -it <pod_name> -- /bin/bash

This command allows you to open a shell inside one of the containers, enabling you to test communication and debug issues.


