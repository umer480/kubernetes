# AKS Networking  - Azure CNI (Container networking interface):

## Azure CNI (Container networking interface):

### Azure CNI key design points:
Azure CNI (Container Networking Interface) is a networking solution for AKS that integrates with Azure Virtual Network (VNET). it allows pods to receive IP address from the Azure VNET, enabling seamless communication between pods and other resources within the VNET.

**Networks** : 

There are 2 networks/CIDR ranges CNI (non overlay):

**1**- **NODE Network/POD Network** (Subnet of VNET where AKS Cluster deployed)  -pods and nodes are in the same network.

**2**- **Cluster Network** (within the cluster) - used with 'Services' to communicate with pods

<img width="1434" height="493" alt="image" src="https://github.com/user-attachments/assets/71854e26-35eb-4533-9128-03e5030094d9" />


**Use Case**:
**Azure CNI is ideal for scenarios where you need**:

**Direct Communication**: pods need to communicate directly with other Azure resources. (VMs, databases) within the same VNET without NAT.
Pods can directly access other Azure resources within the same VNet without needing Network Address Translation (NAT) at the node level.

**Network Security**: Enhanced security through Azure VNET features like Network Security Group (NSG) and Azure Firewall.

**Network policies**:
Azure CNI supports the implementation of Kubernetes Network Policies to control traffic flow between pods.


**Use Azure CNI when**:

- You have sufficient available IP address space.( at VNET level)
- Most of the pod communication is to resources outside of the cluster.
- You don't want to manage user-defined routes for pod connectivity.
- You need AKS advanced features, such as virtual nodes or Azure Network Policy.

**Scability**:
Limited by the number of IP addresses available i the VNET subnet.

**IP address planning**:
This model requires careful planning of IP address ranges within your VNet to accommodate the number of nodes and pods you anticipate in your AKS cluster, as each pod consumes an IP address.


**Designing thing** !!! :
`NODE/POD Network shouldn't overlap with other networks - other VNETs (Spokes) , On-Premise.`



### Understand Traffic Flow by a Real Architecture:

**Egress**: How does PODS goes outside?

**Ingress**: How traffic reach on POD from outside/internet.

✔ Classic HUB SPOKE Architecture. \
✔ VNET Peering.\
✔ VPN Gateway - S2S VPN Tunnel.\
✔ Azure Firewall.\
✔ Multi-Node AKS Cluster with VNET integration.\
✔ VNET integration. \
✔ External/Public and Internal/Private LoadBalancer.



<img width="1427" height="753" alt="image" src="https://github.com/user-attachments/assets/4b0b6399-1f5c-4a67-a2ed-838d89d80e96" />


### LAB : deploy an application and expose it via external and internal service.

Reference: https://www.youtube.com/watch?v=EoLa-1ra15w


**CNI Validation Points**:

✔ Validate pods have the same IP from the node network range?  `#ifconfig` \
✔ Validate ELB can access services running on pods ( via both internal+external LB) ? `# http://<ELB-IP> ( ELB IP > NODE PORT > CLUSTER IP > POD )` \
✔ Validate POD outbound/internet IP address (it should be the same as  ELB has) ? `#curl https://ifconfig.me`


