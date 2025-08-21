# How to connect to AKS Cluster using AZ CLI

```bash
az account show
az account list -o table


az login
az account set --subscription "<subscription-name-or-id>"
az aks get-credentials --resource-group <resource-group-name> --name <aks-cluster-name>
```

### validation

```bash
kubectl get nodes
```

### AKS Set/Change Context

```bash
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl delete-context <content name>
```
