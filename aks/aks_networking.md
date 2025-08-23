# AKS Networking

## Topics:

- AKS Networking Models/Plugins (Kubenet/CNI/CNI Overlay)
- Network Architecture - Hub-Spoke Architecture.
- Traffic Flow ingress and egress.
- Internal and External LoadBalancer.
- Application Gateway. AGIC Controller
- Managed and Unmanaged Nginx Controller.
- Nginx DNS and Key Vault integration. 
- Web Application Firewall / Layer7 Advanced/Smart Routing decisions.
- Front Door + AKS for global ingress
- Private Cluster (API Server only via private endpoint)


## Kubernetes basic networking and communication:

1- Container-to-Container Networking
2- Pod-to-Pod Networking
3- Pod-to-Service Networking
4- Internet-to-Service Networking


1- **Container Communication within a Pod (container-to-container)**

- **Shared Network Namespace**: Containers within the same pod share the same network namespace, meaning they can communicate with each other via localhost and share the same IP address and port space.
- **Inter-Process Communication (IPC)**: Containers in a pod can use standard IPC mechanisms like SystemV semaphores or POSIX shared memory to communicate.
- **Shared Volumes:** Containers in the same pod can also communicate by reading and writing to shared volumes.


2- **Pod Communication (pod-to-pod)**

**IP-Per-Pod Model:** Each pod in Kubernetes is assigned a unique IP address, allowing direct communication between pods without the need for Network Address Translation (NAT). This simplifies networking and ensures that pods can easily find and talk to each other across the cluster.

**Kube-proxy**: This component runs on each node and manages network rules to allow communication between pods. It handles routing and load balancing for services within the cluster.


<img width="1100" height="738" alt="image" src="https://github.com/user-attachments/assets/f72dead7-bc00-4cde-b538-7c35b8a734ef" />



3- **Pod-to-Service Networking**

**Stable IP Addresses**: Services provide stable IP addresses or hostnames for accessing a set of pods, ensuring consistent access even if individual pods change.

**EndpointSlice Management:** Kubernetes automatically manages EndpointSlice objects to track the pods backing a service, enabling efficient load balancing and service discovery.


<img width="1100" height="1117" alt="image" src="https://github.com/user-attachments/assets/dc0c5dfe-2501-4f2b-b4f6-4085753b1ddf" />

4- **Internet-to-Service Networking:**


`A Service allows you to access a group dynamically of deployment (replicaset) pods.`

**Ingress Controllers:** Facilitate Layer 7 routing, enabling SSL/TLS termination and routing based on hostnames or paths.

**Load Balancers:** Distribute incoming traffic across multiple backend pods, ensuring high availability and reliability. Azure automatically configures network security group rules and manages DNS settings for HTTP application routing when new Ingress routes are established.


<img width="1100" height="1147" alt="image" src="https://github.com/user-attachments/assets/a3acf4b6-ecd1-4c45-8733-fdf1c424e1be" />


### Key Points:

<img width="881" height="662" alt="image" src="https://github.com/user-attachments/assets/6be80a7a-29a1-4433-a3f2-65201945fff4" />


### Container networking:

Kubernetes leverages Container `Networking Interface (CNI) plugins` to handle networking within its clusters. These CNIs are tasked with assigning IP addresses to pods, managing network routing between pods, and handling Kubernetes Service routing, among other functions.

**Azure Kubernetes Service (AKS) offers a variety of CNI plugins to suit different networking needs in your clusters:**

1- **Azure CNI overlay (overlay network model)**: Private and scalable.\
2- **Azure CNI Node Subnet (flat network model):** Use it when AKS needs to communicate with other Azure resources.\
3- **kubenet:** Basic networking option for Azure Kubernetes Service (AKS). Not recommended.\



### Understand Communication Flows:

```bash
Within POD Communication
Pod Communication (pod-to-pod)
Pod-to-Service Networking
Internet-to-Service Networking
```

**In other words:**

```bash
- Communication from/to the internet - External Load Balancer (ELB)
- Communication from/to VNET
- Communication from/to On-Premise  -Internal Load Balancer (ILB)  -VNET Peering - S2S VPN Tunnel
- Communication with Azure Services that have Private EndPoints.
```


### Understand Traffic Directions:
 
 🔹 Egress vs Ingress in AKS (and networking in general)


```bash

Egress traffic → traffic leaving a Pod/node/cluster (outbound).

Ingress traffic → traffic arriving into a Pod/node/cluster (inbound).
```



# Networking Models / Network Modes / Network Plugins - in  AKS

## What is networking models in AKS:

The networking model in AKS (Azure Kubernetes Service) defines how Pods get their IP addresses and how they talk to each other, to the node, to the internet, and to your VNET.

`Think of it as the rules of the road for traffic inside and outside the cluster.`


1- kubenet. (basic, legacy) --> deprecated) ****  \
2- CNI Overlay  - CNI Pod Subnet \
3- CNI (non Overlay) - CNI Node Subnet \





Reference: 
```bash
https://medium.com/@h.stoychev87/azure-aks-network-components-part-1-2025-edition-ce5439f4c767
```










