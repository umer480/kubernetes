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
    app: test
  ports:
    - port: 80            # Service port
      targetPort: 8080    # Pod's container port

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




# Create or Selection of internal/public load balancer on Azure :

Use **annotation**:


```bash

service.beta.kubernetes.io/azure-load-balancer-internal: "true"

service.beta.kubernetes.io/azure-load-balancer-internal: "false"   **(default)**


```

```bash

apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"   # set true/false
spec:
  type: LoadBalancer
  selector:
    app: test
  ports:
  - port: 80           # service port ( same for loadbalacner)
  - targetPort: 80     # Pod's container port

```

# Use Specific or Custom IP Address (Custom frontend ip )

Manually provision ip address resource on azure portal.
and then set ip address in annotation; as **below**

```bash

  annotations:
    service.beta.kubernetes.io/azure-load-balancer-ipv4: "52.191.99.127

```


```bash

apiVersion: v1
kind: Service
metadata:
  name: svc-custom-ip
  namespace: test
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-ipv4: "52.191.99.127"    # provide ip address here
spec:
  type: LoadBalancer
  selector:
    app: test
  ports:
  - port: 80   # this will act as external port - means loadbalancer port (same service port)
    targetPort: 80

```

# Restrict inbound traffic to specific IP ranges
The following manifest uses loadBalancerSourceRanges to specify a new IP range for inbound external traffic.

```bash
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
  loadBalancerSourceRanges:
  - MY_EXTERNAL_IP_RANGE

```

**NOTE:******

This example updates the rule to allow inbound external traffic only from the MY_EXTERNAL_IP_RANGE range. If you replace MY_EXTERNAL_IP_RANGE with the internal subnet IP address, traffic is restricted to only cluster internal IPs. If traffic is restricted to cluster internal IPs, clients outside your Kubernetes cluster are unable to access the load balancer.

![image](https://github.com/user-attachments/assets/50c56f90-b23c-4141-b5d2-62ba9d18dbad)




# Session Affinity / Persistence: 

```bash
sessionAffinity: None
```


```bash

apiVersion: v1
kind: Service
metadata:
  name: svc-session-persistence
spec:
  type: LoadBalancer

  sessionAffinity: ClientIP  # Enables session persistence based on Client IP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3 hours (10800 seconds) of session stickiness

  selector:
    app: test
  ports:
  - port: 80           # service port ( same for load balancer)
    targetPort: 80     # Pod's container port

```


# Maintain the client's IP on inbound connections

By default, a service of type LoadBalancer in Kubernetes and in AKS doesn't persist the client's IP address on the connection to the pod. The source IP on the packet that's delivered to the pod becomes the private IP of the node. To maintain the client’s IP address, you must set service.spec.externalTrafficPolicy to local in the service definition. The following manifest shows an example.


`externalTrafficPolicy: Local`

```bash

apiVersion: v1
kind: Service
metadata:
  name: svc-preserve-clientip
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local      # <-- This preserves the client IP
  ports:
  - port: 9090
  selector:
    app: test
```

If you set 'externalTrafficPolicy: Local' then you cross node loadbalancing will be disable - you can reach only local pods - if any node does not have pod then traffic will be dropped. 


# Service in Separate namespace:

```bash

apiVersion: v1
kind: Service
metadata:
  name: svc-test-ns
spec:
  type: LoadBalancer
  selector:
    app: test
  ports:
  - port: 80           # service port ( same for loadbalacner)
    targetPort: 80     # Pod's container port

```
**Question:**
i have a pods deployment in default namespace with label `app: test` - and if i create a load balancer service in a seperate namespace  and use `app: test` in selector then pods will be added as endpoints for LB service?

`No`, the LoadBalancer service in a separate namespace will not be able to select the pods from the default namespace even if they share the same label (app: test).


**🔎 Why?**
In Kubernetes, Services are namespace-scoped.

A Service in one namespace can only discover and select Pods in the same namespace.

`Even if the labels match, Kubernetes will not cross namespaces to bind the Service to the Pods.`





# Delete the load balancer:

The load balancer is deleted when all of its services are deleted.

As with any Kubernetes resource, you can directly delete a service, , which also deletes the underlying Azure load balancer.


`test`
