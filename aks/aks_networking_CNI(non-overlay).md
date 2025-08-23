# AKS Networking  - Azure CNI (Container networking interface):

'In this model, each pod receives an IP address directly from the Azure Virtual Network (VNet) subnet where the AKS cluster is deployed.'

### Azure CNI key design points:
Azure CNI (Container Networking Interface) is a networking solution/model for AKS that integrates with Azure Virtual Network (VNET). it allows pods to receive IP address from the Azure VNET, enabling seamless communication between pods and other resources within the VNET.

**Flat Network:**

In Azure `CNI Pod Subnet` (a type of flat network), both nodes and pods receive IP addresses directly from your virtual network (VNet). This necessitates a larger VNet subnet compared to overlay networks. To accommodate this, you must meticulously plan for the maximum number of nodes and pods your cluster will require. Additionally, because nodes and pods utilize separate subnets within your VNet, you must plan and allocate IP ranges for both independently.


### Simplest Diagram of how pods connect with the node network ( via Bridge):

Instead of NAT, Azure CNI creates a bridge for the Pod to be directly visible inside the Vnet. There is no NAT so no additional hop, which means performance similar to VM to VM communication.

Bridge = virtual switch  ->It connects multiple network interfaces (pods)  so they can talk at Layer 2 (Ethernet).

<img width="829" height="481" alt="image" src="https://github.com/user-attachments/assets/700baa31-1d7b-4dff-99dd-77e0b8a13ed1" />


**Networks/CIDR** : 

`There are 2 networks/CIDR ranges CNI (non overlay):`

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


### Notes:

> [!IMPORTANT]

> [!NOTE]      - Load balancer talks to the nodes - not pods -- LB has health probes that point/monitor nodeport of nodes.  
> [!NOTE]      - Internal LB does not have outbound rules - it's applicable only for EL. 
> [!CAUTION]   - Don't remove the Outbound rule from the external LB; otherwise, pods will not be able to reach the internet / outbound internet connectivity will stop. 


**CNI Validation Points**:

✔ Validate pods have the same IP from the node network range?  `#ifconfig` \
✔ Validate ELB can access services running on pods ( via both internal+external LB) ? `# http://<ELB-IP> ( ELB IP > NODE PORT > CLUSTER IP > POD )` \
✔ Validate POD outbound/internet IP address (it should be the same as  ELB has) ? `#curl https://ifconfig.me`




#  **Key Difference with kubenet**

| **Model/Plugin**            | **Pod-to-Pod (east-west)**                        |                 **Pod-to-Outside (north-south)**                                                           
| --------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------  |
| **kubenet**                 | Needs **NAT or UDR-based routing** for cross-node | Always **SNATs to Node IP** before leaving                             |
| **Azure CNI Overlay**       | Uses **VXLAN tunnels**, **no NAT** inside cluster | **SNAT to Node IP** (or NAT Gateway/SLB IP)                            |
| **Azure CNI (non-overlay)** | Direct **Pod IP routing (VNET IPs)**, no NAT      | No NAT needed (Pod IPs are valid in VNET) unless outbound rules apply  | 

