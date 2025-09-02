# Switch to another Cluster


### Set Subscription:

```bash
az account set --subscription 00351350-14ae-465d-9454-63727cda9c24
```

### get current configs:
```bash
kubectl config view
kubectl config current-context

```
### change/switch cluster
```bash
kubectl config get-contexts
kubectl config use-context <context-name>

```
