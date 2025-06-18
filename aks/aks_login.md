# How to connect to AKS Cluster using AZ CLI

```bash
az login
az account set --subscription "<subscription-name-or-id>"
az aks get-credentials --resource-group <resource-group-name> --name <aks-cluster-name>
```

### validation

``bash
kubectl get nodes
```
