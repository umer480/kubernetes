# Provision AKS Cluster

- Managed Control Plane. (We can't access the master nodes )
- VMSS - worker nodes


### Pre-set configurations comparison:

- Bring your own VNET - Bringing your own Azure Virtual Network (VNet) to an Azure Kubernetes Service (AKS) cluster allows for greater control over network topology.IP address management, and integration with existing network infrastructure.
- node auto provisioning (NAP).
- Enable Virtual Node.
- Enable Private Cluster
- Enable Node Public IP?
- Authorized Public IPs? IP Whitelisting
- Network Plugin / Network Model Selection *****  (Azure CNI Overlay and Azure CNI Node Subnet)
- Service mesh - Istio
- Network Policy (Calico and Azure native)
- Azure Policy
- Azure key Vault integration

- 

<img width="1942" height="1266" alt="image" src="https://github.com/user-attachments/assets/409b09ea-ea69-40cd-8169-5f8585cb669e" />



### User User-assigned managed identity (MSI) in AKS:


When you create an AKS cluster, Azure automatically creates a User-Assigned Managed Identity (or a Service Principal, though Managed Identity is now the default and recommended practice).

The purpose of this cluster identity is to authorize the AKS control plane (the cluster itself) to interact with Azure APIs on your behalf.


Think of it as the "cluster's identity card" for talking to other Azure services. It is used by the core Azure platform components that manage your cluster


**Managing Load Balancers**: When you create a Kubernetes LoadBalancer service, the AKS control plane uses this identity to call the Azure Networking API to create and configure a real Azure Load Balancer.

**Managing Storage**: When you create a PersistentVolumeClaim for an Azure Disk or Azure File share, the AKS control plane uses this identity to create and manage those storage resources in your resource group.

**Managing Container Registries**: It is used to pull container images from Azure Container Registry (ACR) if you attach the ACR to the cluster.


### Virtual Node:

Reference: https://www.youtube.com/watch?v=LhOCFJZp1H0

handles pending pods/unsechedulanble pods

`AKS+ACI (Azure Container instance)`:

<img width="907" height="410" alt="image" src="https://github.com/user-attachments/assets/a0e77e7e-2485-4118-a46d-b8f976cbe065" />


<img width="1015" height="474" alt="image" src="https://github.com/user-attachments/assets/ee2cc723-2ed8-4268-967e-b6af1863763a" />

  Exensive Solution: manually add worker node/VM - or use Horizontal Cluster AutoScaler for more workload - also Adding VM based worker node takes some time to get it up and running.

  Virtual Nodes is useful:
- Handles seasonal burst traffic.
- Cost-effective Solution. ( but may expensive than VMs if you run it for longer time) so use only for spikes/seasonal traffic.
- Serverless - pay per second for ACI.
- Virtual noodes has labels and taint.
- Virtual node has big CPU and Memory limit ( but dont cost actually - only cost occurs when containers runs)


  

