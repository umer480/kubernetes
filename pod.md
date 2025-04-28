# POD 

## What is POD:
A pod is the smallest and simplest Kubernetes object.
POD is nothing but a logical boundary for containers. - POD contains multiple containers - POD is a collection of containers.


**A pod makes sure those containers:**
Share the same network (they can talk to each other easily)
Share storage if needed
Are scheduled together (they run on the same machine)


You usually put one container in a pod. But sometimes, you might put multiple containers that must work closely together.

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
2-Declarative way

## Create POD - Imperative way

kubectl run mypod --image-nginx

# watch command output:

kubectl get pods --watch 


## Create POD - Declarative way ( + Print out ready made YAML)
```bash
kubectl run podname --image=nginx --dry-run=client -o yaml > mypod.yml
```

Each Pod gets its own IP address, allowing communication with other Pods in the cluster
pods are ephemeral by nature, if a pod (or the node it executes on) fails, Kubernetes can automatically create a new replica of that pod to continue operations.





![image](https://github.com/user-attachments/assets/6868fdd8-dfa6-4bae-b294-e35377179641)



![image](https://github.com/user-attachments/assets/ae3cd716-ed2e-4c60-abf1-094103523777)



![pod-animation](https://github.com/user-attachments/assets/35c5d3ae-7671-4702-bb0f-71baa2a8870c)



