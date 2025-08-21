# AKS Networking


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

Three networks/CIDR:

1- NODE Network (Subnet of VNET where AKS Cluster deployed)  172.16.239.0/24
2- Cluster Network (within the cluster) - used with 'Services' to communicate with pods  10.101.0.0/16
3- POD Network 10.244.0.0/16 (within the cluster)


### Azure CNI key design points:
Azure CNI (Container Networking Interface) is a networking solution for AKS that integrates with Azure Virtual Network (VNET). it allows pods to receive IP address from the Azure VNET, enabling seamless communication between pods and other resources within the VNET.

Networks : 
1- NODE Network/POD Network (Subnet of VNET where AKS Cluster deployed)  -pods and nodes are in the same network
2- Cluster Network (within the cluster) - used with 'Services' to communicate with pods

Use Case:
Azure CNI is ideal for scenarios where you need:

Direct Communication: pods need to communicate directly with other Azure resources. (VMs, databases) within the same VNET without NAT

Network Security: Enhanced security through Azure VNET features like Network Security Group (NSG) and Azure Firewall.

Scalinility:
Limited by the number of IP addresses available i the VNER subnet


NODE/POD Network shouldnt be overalap with other networks from - IP overlapping issues, with peering on-premise.




<img width="1099" height="746" alt="image" src="https://github.com/user-attachments/assets/9fa431c2-b698-4aa1-b657-ce20283d3c3c" />


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
