
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





### Azure DNS Zone integration:

```bash
Reference : https://learn.microsoft.com/en-us/azure/aks/app-routing-dns-ssl#create-a-public-azure-dns-zone
```


```bash
az aks approuting zone add --resource-group <ResourceGroupName> --name <ClusterName> --ids=${ZONEID} --attach-zones

az aks approuting zone add --resource-group rg-aks-demo --name aks-demo-cluster --ids=/subscriptions/62efaa2d-1d10-400c-bbe5-b8890978d790/resourceGroups/rg-dns-demo/providers/Microsoft.Network/dnszones/cloudlyncs.com --attach-zones

```

Example of ingress:

`Make sure cloudlyncs.com DNS Zone exists`.

```bash

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: www.cloudlyncs.com
  namespace: aks-store

spec:
  ingressClassName: webapprouting.kubernetes.azure.com
  rules:
  - host: www.cloudlyncs.com
    http:
      paths:
      - backend:
          service:
            name: store-front
            port:
              number: 80
        path: /
        pathType: Prefix


```


### Key Vault integration for SSL/TLS termination

You command below to connect Azure KV with the ingress managed controller/app routing.

```bash
az aks approuting zone add --resource-group <ResourceGroupName> --name <ClusterName>  --enable-kv --attach-kv <KV-ID>

az aks approuting update --resource-group rg-aks-demo --name aks-demo-cluster --enable-kv --attach-kv /subscriptions/62efaa2d-1d10-400c-bbe5-b8890978d790/resourceGroups/rg-aks-demo/providers/Microsoft.KeyVault/vaults/kv-aks-ingress-demo

```


**Example**:

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: www.cloudlyncs.com
  namespace: aks-store
  annotations:
    kubernetes.azure.com/tls-cert-keyvault-uri: https://kv-aks-ingress-demo.vault.azure.net/secrets/www-cloudlyncs-com/a84a9f593abd4023a930f2894864bcbf

spec:
  ingressClassName: webapprouting.kubernetes.azure.com
  rules:
  - host: www.cloudlyncs.com   # <hostname>
    http:
      paths:
      - backend:
          service:
            name: store-front
            port:
              number: 80
        path: /
        pathType: Prefix

  tls:                        # this will open port 443 for ingress and fetch the SSL Cert from KV certificate.
  - hosts:
    - test.cloudlyncs.com     # <hostname>
    secretName: keyvault-www.cloudlyncs.com    # keyvault-<ingress name>  that you define ^ under  metadata.name
```



Commands
```bash

kubectl get ingress
kubectl get ing
kubectl get ing -A | grep www.cloudlyncs.com    --> use grep to get specific ingress
kubectl get ing,ingressclass -n <name space>    -> get ingress and ingressclass of a specific name space in a single command 
```

===AKS Default SLB : provides outbound/internet access to pods.===

AKS clusters with a default standard load balancer provide outbound access to pods. When an AKS cluster is created with a standard load balancer, a public IP address is automatically provisioned and associated with the load balancer's outbound pool, enabling pods to access external resources.

When you create 'ingress' then it also creates a ConfigMap.
