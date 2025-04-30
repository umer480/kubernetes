# POD 

## What is POD:
A **pod** is the smallest and simplest Kubernetes object that you create and manage in kubernetes.

**POD** is nothing but a logical boundary for containers. 
- POD contains multiple containers
- You can say POD is a collection of containers.


# POD Very Basic Diagram:

![image](https://github.com/user-attachments/assets/ab37b00e-5762-45be-afff-5e1a6bbb6dce)


**A pod makes sure those containers:**
-Share the same network (means - same IP address, use the same network  interface)
-Share storage if needed
-are scheduled together (they run on the same machine)


You usually put **one container** in a pod. But sometimes, you might put multiple containers that must work closely together.

```bash
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
```

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
1- imperative way.

2- Declarative way.

## Create POD - Imperative way

```bash
kubectl run mypod --image-nginx
```

## Create POD - Declarative way ( + Print out ready made YAML)
```bash
kubectl run podname --image=nginx --dry-run=client -o yaml > mypod.yml
```






## Communication within same pod / communication between multiple containers s within same POD:

## Shared Network Namespace
One of the key features that enable container communication within a Pod is the shared network namespace. This means that all containers within a Pod share the same network resources, such as IP address, network interfaces, and ports. This shared environment allows containers to communicate with each other directly using localhost or 127.0.0.1 as if they were running within the same operating system instance.

**Loopback Interface**: Containers within a Pod can communicate via the loopback interface, using the localhost address. For example, if one container is running a service on port 8080, another container within the same Pod can access this service by connecting to localhost:8080.

**Shared IP Address:** Every Pod is assigned a unique IP address within the Kubernetes cluster, and all containers in the Pod use this IP. This eliminates the need for complex networking configurations between containers in the same Pod.


**Example : communication via localhost of web and db server**
consider a scenario where one container in a Pod runs a **web server on port 8080**, and another container runs a database server on port 3306. The web server can connect to the database using **localhost:3306**, leveraging the shared network namespace.

![pod-animation](https://github.com/user-attachments/assets/35c5d3ae-7671-4702-bb0f-71baa2a8870c)





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
```bash
![image](https://github.com/user-attachments/assets/ae3cd716-ed2e-4c60-abf1-094103523777)
```




## PODs - Commands
```bash
kubectl get pods
kubectl get pods -A   --> list pods in all name spaces
kubectl get pods -n <namespace>   -> list pod of specific namespace
kubectl get pods -o wide  --> wide view - to see pod's ip address / to see on which node pod is running
kubectl describe mypod
kubectl delete pod mypod

**## POD Creation  (not recommended for production):**
kubectl run <pod-name> --image=<image-name>
kubectl apply -f <pod-definition.yaml>
kubectl apply -f <pod.yaml> --dry-run=client    ----> **Dry run Pod creation (only show what would happen)**

```




## See POD logs - for Troubleshooting
```bash
kubectl logs <pod-name>  --> view pod logs
kubectl logs <pod-name> -c <container-name>   -->View logs of a specific container inside a Pod
kubectl logs -f <pod-name>   -->Stream (follow) Pod logs:
kubectl get events  -->See all cluster events




****# Access a Container Within a Pod:****
This command allows you to open a shell inside one of the containers, enabling you to test communication and debug issues.
```bash
kubectl exec -it <pod_name> -- /bin/bash
```


# watch command output:
```bash
kubectl get pods --watch
```

# Port-forward from your local machine to a Pod:
```bash
kubectl port-forward pod/<pod-name> <local-port>:<pod-port>
```


## Restart a POD:
In Kubernetes, you cannot directly restart a Pod because a Pod is meant to be ephemeral — if you need to restart it, you usually delete it and Kubernetes (or your controller like Deployment, ReplicaSet, etc.) will recreate it.



## Summary: What you get ? note down these important key points

![image](https://github.com/user-attachments/assets/410ca112-690d-48f8-9ade-8ac3f2acd3af)

### LABS Scenarios:
- Create/Deploy a pod (via command / via yml)
- How to deploy pod with one or more containers
- Deploy web+db on same pod ( multi container = 1 for web, 1 for db)
- Deploy 2 tier application (web+db) - 2 pods = 1 for web,1 for db



## Pod Redundancy & Recovery: Because Kubernetes assigns a new IP from the node's Pod CIDR range every time a Pod is (re)created.
**-** If the container inside the Pod crashes (e.g., due to an app crash), the kubelet may restart the container based on the restartPolicy (default is yes)
**-** A standalone Pod **(kind: pod)**  will not be automatically recreated if it's deleted manually or if the node it's running on goes down.
**-** Pod IP address will change: When a Pod is deleted and recreated. Because Kubernetes assigns a new IP from the node's Pod CIDR range every time a Pod is (re)created.


**So, you should not deploy standalone pods using (kind: pod) -** 
no Redundancy ![image](https://github.com/user-attachments/assets/d69976ab-aa98-4f32-bcde-4db3494391a4)

no Load balancing ![image](https://github.com/user-attachments/assets/029d77e4-997f-45b6-a14a-bf233e99d0eb)
 
no fault tolerance ![image](https://github.com/user-attachments/assets/b6b849a0-b8c9-4d65-9386-c228964fb777)

no reliable connectivity ![image](https://github.com/user-attachments/assets/5414f03b-7724-499b-8d61-d4b04784e650)



## SOLUTION:
To overcome above mentioned issues:
Always use "**Deployments**" and "**Service**" which provide redundancy and reliable network connectivity.


