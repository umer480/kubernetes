
# Managed Nginx Controller on AKS
## App Routing Feature of AKS


When you enable app routing on AKS then it creates  2 types of controllers: `Nginx controller` and `External DNS Controller`

<img width="923" height="404" alt="image" src="https://github.com/user-attachments/assets/03405bb0-15d8-47ca-b7f9-65e87860baa8" />


### Enable App Rputing on AKS Cluster

```bash
az aks approuting enable --resource-group rg-aks-demo --name aks-demo-cluster
```





### Traffic Flow:

```bash
Public/interet User --> External Load Balancer (Public IP) <-- LB Service ---> Ingress Controller <-- ingress (Sync) --> Service(ClusterIP) --> POD
```






===AKS Default SLB : provides outbound/internet access to pods.===

AKS clusters with a default standard load balancer provide outbound access to pods. When an AKS cluster is created with a standard load balancer, a public IP address is automatically provisioned and associated with the load balancer's outbound pool, enabling pods to access external resources.

When you create 'ingress' then it also creates a ConfigMap.
