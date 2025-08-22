# AKS Networking

Reference:
https://medium.com/@h.stoychev87/azure-aks-network-components-part-1-2025-edition-ce5439f4c767


# Deploy AKS on Existing Environment.

### Brownfield environment:

brownfield network environment in networking is a previously developed network or infrastructure that is modified, upgraded, or integrated with new technologies and systems, rather than being built from scratch.

Classic Hub-Spoke architecture:


Network Models: IP Address Management

Networking Models - define how pods are communicate with each other and the external world.
Network Designing:

1-Kubenet
2-CNI
3- CNI Overlay


### Kubenet

pods receive IP addresses from a logically separate address space, and traffic is NATed to the node's IP address when leaving the node

Three networks/CIDR:

1- NODE Network (Subnet of VNET where AKS Cluster deployed)  172.16.239.0/24
2- Cluster Network (within the cluster) - used with 'Services' to communicate with pods  10.101.0.0/16
3- POD Network 10.244.0.0/16 (within the cluster)


NAT is performed.
An additional hop is required in the design of kubenet, which adds minor latency to pod communication.
Route tables and user-defined routes are required for using kubenet, which adds complexity to operations

<img width="780" height="349" alt="image" src="https://github.com/user-attachments/assets/0384632f-9b70-4776-b5b7-ac35e7b1097c" />


**Use kubenet when**:

You have limited IP address space.
Most of the pod communication is within the cluster.
You don't need advanced AKS features, such as virtual nodes or Azure Network Policy.

### Azure CNI key design points:
Azure CNI (Container Networking Interface) is a networking solution for AKS that integrates with Azure Virtual Network (VNET). it allows pods to receive IP address from the Azure VNET, enabling seamless communication between pods and other resources within the VNET.

**Networks** : 
1- NODE Network/POD Network (Subnet of VNET where AKS Cluster deployed)  -pods and nodes are in the same network
2- Cluster Network (within the cluster) - used with 'Services' to communicate with pods

<img width="1434" height="493" alt="image" src="https://github.com/user-attachments/assets/71854e26-35eb-4533-9128-03e5030094d9" />

**Use Case**:
**Azure CNI is ideal for scenarios where you need**:

**Direct Communication**: pods need to communicate directly with other Azure resources. (VMs, databases) within the same VNET without NAT.
Pods can directly access other Azure resources within the same VNet without needing Network Address Translation (NAT) at the node level.

**Network Security**: Enhanced security through Azure VNET features like Network Security Group (NSG) and Azure Firewall.

**Network policies**:
Azure CNI supports the implementation of Kubernetes Network Policies to control traffic flow between pods.

**Use Azure CNI when**:

You have available IP address space.
Most of the pod communication is to resources outside of the cluster.
You don't want to manage user defined routes for pod connectivity.
You need AKS advanced features, such as virtual nodes or Azure Network Policy.

**Scability**:
Limited by the number of IP addresses available i the VNER subnet
**IP address planning**:
This model requires careful planning of IP address ranges within your VNet to accommodate the number of nodes and pods you anticipate in your AKS cluster, as each pod consumes an IP address.



NODE/POD Network shouldnt be overalap with other networks from - IP overlapping issues, with peering on-premise.




<img width="1099" height="746" alt="image" src="https://github.com/user-attachments/assets/9fa431c2-b698-4aa1-b657-ce20283d3c3c" />


<img width="1446" height="674" alt="image" src="https://github.com/user-attachments/assets/f6d9ecc2-6458-414d-9221-39d36b44b731" />





## Azure CNI Overlay 

### Communication directions/flow:

`Pod IPs are not NATed at all inside the cluster.`

When Pod A talks to Pod B (even across nodes), the original Pod IP is preserved end-to-end.

`The only “trick” is that the Pod IP ranges are not part of the VNET → so Azure CNI overlay uses VXLAN encapsulation between nodes to carry that Pod-to-Pod traffic.`



🔹 **1- East-West (inside the cluster):**

**Pod ↔ Pod on same node** → They just use the overlay bridge, no NAT.

**Pod ↔ Pod across nodes** → Overlay (VXLAN) tunnels the traffic between nodes. The original **Pod IPs remain intact**.

👉 So **within the cluster, Pods always talk to each other directly with Pod IPs (no NAT involved).**





🔹 **2-  North-South (outside the cluster / VNET / internet):**

When a Pod sends traffic outside the cluster overlay (e.g., internet, Azure SQL, Storage Account, on-prem):

The Pod IP **is not routable outside the overlay.**

The node’s host network does **SNAT **the Pod IP → node IP (or NAT Gateway/SLB outbound IP).

👉 So the **outside world never sees the Pod IP** — it only sees the node/NAT IP.




#  **Key Difference with kubenet**

| **Model/Plugin**            | **Pod-to-Pod (east-west)**                        |                 **Pod-to-Outside (north-south)**                                                           
| --------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------  |
| **kubenet**                 | Needs **NAT or UDR-based routing** for cross-node | Always **SNATs to Node IP** before leaving                             |
| **Azure CNI Overlay**     | Uses **VXLAN tunnels**, **no NAT** inside cluster | **SNAT to Node IP** (or NAT Gateway/SLB IP)                            |
| **Azure CNI (non-overlay)** | Direct **Pod IP routing (VNET IPs)**, no NAT      | No NAT needed (Pod IPs are valid in VNET) unless outbound rules apply  | 


Virtual Network (VNet) Integration


**Service Exposure & Load Balancing**:

ClusterIP → Internal-only service (default).

NodePort → Exposes service on each node’s IP:Port (not usually recommended for production).

LoadBalancer → Creates an Azure Load Balancer (public or internal) and assigns external IP.

Ingress Controller → Provides Layer 7 routing (HTTP/HTTPS) using NGINX, App Gateway, etc.

**Ingress with Application Gateway (AGIC)**

App Gateway Ingress Controller integrates AKS with Azure Application Gateway.

Provides WAF, SSL termination, and Layer 7 routing.

Alternative to NGINX ingress when you need enterprise-grade features.


**DNS and Service Discovery**

CoreDNS handles DNS inside the cluster.

Services are discoverable via <service-name>.<namespace>.svc.cluster.local.

Can resolve external DNS via upstream resolvers.


### Outbound (Egress) Traffic
### Private Clusters
