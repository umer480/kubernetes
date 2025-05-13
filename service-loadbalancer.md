# LoadBalancer - Service

A LoadBalancer is also a type of service that exposes the pod to external traffic. As the name implies, a LoadBalancer service distributes the traffic between the nodes/pods that are targeted by the service.


- **LoadBalancer Service type is related to Cloud provider** - When we use a Managed Cluster ( Managed Control plane) i.e  ( Azure AKS, AWS EKS, GCP GKE)

# External LoadBalancer : (Outside Kubernetes Cluster)
When we create a LB service then it **Automatically provisions an external load balancer**  (usually in a cloud environment like AWS, Azure, etc.) to expose the service outside the cluster to access from the internet.

this load balancer is provision by the controller of cloud provider  that resides in the kubernetes cluster . it interect with the cloud provider and manage cloud related actions. ( create LB, add routing rules , backend endpoints  etc)


# Why we need LoadBalancer Type Service:???
**1- Security Concern**-   NodePort services do not provide the same level of network security as LoadBalancer services, as they expose the service directly to the node.


**2- No Node-Level Redundancy/LoadBalancing:** Kubernetes doesn’t automatically load balance incoming traffic across nodes.You must handle that externally if you want high availability or balanced traffic at the node level.

**Key-Concept:**
- kube-proxy ensures the NodePort on any node can route to any pod in the cluster, not just local pods.
- Without external loadbalacner This is just cluster-level load balancing to pods, not node-level load balancing from outside the cluster.


![image](https://github.com/user-attachments/assets/e2357b3e-26b4-41ae-b168-dabbb0d649be)

# **When you create a LoadBalancer type service in Kubernetes, the behavior is as follows:**

**✅ 1️⃣ External Load Balancer Creation:**
Kubernetes interacts with your cloud provider (e.g., AWS, Azure, GCP) to create an external Load Balancer.

This Load Balancer gets a public IP address and exposes your service to the outside world.

**✅ 2️⃣ Endpoints of the Load Balancer:**
The external Load Balancer (managed by the cloud provider) routes traffic to all worker nodes in your cluster.

It does not route directly to Pods; it routes to the nodes.

**✅ 3️⃣ NodePort Mechanism Behind the Scenes:**
When you create a LoadBalancer service, it automatically creates a NodePort on each worker node.

The Load Balancer forwards traffic to the NodePort of each node.

Kubernetes' internal kube-proxy handles the traffic and forwards it to one of the matching Pods based on the service selector.


**✅ 4️⃣ Traffic Flow Summary:**

```bash
Client Request ➡️ External Load Balancer ➡️ NodePort on Worker Nodes ➡️ kube-proxy ➡️ Matching Pods

```


![image](https://github.com/user-attachments/assets/fe71d6e1-701e-4104-aae7-7dd29e55a5f9)



# Examples: Create LoadBalancer type Service

# Create LoadBalancer Service via yml


```bash
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

**In this example,** the LoadBalancer service named my-service will expose a service running on port 80 and target port 8080. The service will balance the load between pods labeled with app: my-app.

Once you create this service, Kubernetes will automatically create a load balancer in your cloud provider and configure it to forward traffic to the pods in your service. The service can then be accessed from the external network using the IP address of the load balancer.

![image](https://github.com/user-attachments/assets/3956d8c8-d686-4f8d-9cb4-7a4fdee34b20)

You have a limited budget: LoadBalancer is often more expensive than NodePort, as it requires a load balancer in the cloud provider. If you have a limited budget, NodePort is a more cost-effective solution.


**Example:**

```bash

apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80         # Service port
      targetPort: 8080 # Pod's container port

```
**Explanation:**

1- The cloud provider provisions an external Load Balancer.

2- It forwards traffic to port 80 on all worker nodes.

3- Each node has a NodePort exposed.

4- Traffic reaches the Pod on port 8080.



A LoadBalancer service is built on top of a NodePort service, and the cloud provider's load balancer will often map traffic from its own IP to a specific node port on the worker nodes


While a NodePort is involved, you may not always need to explicitly specify one in your LoadBalancer service configuration. The cloud provider might automatically assign one




# Loadbalancer Service ( Uses Nodeport+ClusterIP) :

![image](https://github.com/user-attachments/assets/4afcb971-9985-4b3a-8a9a-ec34c2fec747)


![image](https://github.com/user-attachments/assets/cdee43c4-05d5-42d9-8e21-bdaf7fb32908)



# Key Points:

1- LoadBalancer service is an extension of nodeport

2- nodeport service is an extension of ClusterIP.




# Create a Seperate LB on Azure :

Use **annotation**:

```bash

apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: internal-app
```


# Restrict inbound traffic to specific IP ranges


# Maintain the client's IP on inbound connections


**Reference :** https://learn.microsoft.com/en-us/azure/aks/load-balancer-standard

