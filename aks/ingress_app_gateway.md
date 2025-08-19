# Deploy Ingress controller (AGIC) with Azure Application Gateway - AKS

### Scenarios :

Implementation Scenarios:

1-  Existing AKS Cluster with new Application Gateway.  \
2-  Existing AKS Cluster with existing Application Gateway. \
3-  App Gateway (Public) --> LB (Private) --> Nginx Managed Controller > Cluster IP/Service > POD
4-  LB (Public) --> LB Service >  Nginx Managed Controller > Cluster IP/Service > POD



## LAB Existing AKS Cluster with new Application Gateway:

Steps:
- Enable Application Gateway Addon on AKS Cluster --> it will create a POD for AGIC that will provision App Gateway on Azure.
- Grant ManagedIdentity Network/Contributor Role to VNET that is integrated with AKS `(az aks show -g rg-aks-demo -n aks-demo-cluster --query "addonProfiles.ingressApplicationGateway.identity.clientId" -o tsv)`
- Validate Logs of AGIC POD : `kubectl logs <pod name> -n kube-system`
- Deploy a Sample application and validate its access via App Gateway IP Address

  === Deploy Sample application and access via App Gateway/ingress===


```bash
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml
```

```bash
kubectl get ingress
```



# Managed Ingress App Routing  -Nginx Controller

Enable App Routing on AKS Cluster:
```bash
az aks approuting enable --resource-group rg-aks-demo --name aks-demo-cluster --nginx
```


```bash
https://learn.microsoft.com/en-us/azure/aks/app-routing
```



it will show 'app gateway' ip address if everything is fine.


LAB Scenarios:
AKS with AGIC and App Gateway -  manual configs get oeverrides using AGIC/ingress , downtime each time while deploying changes to app gateway. 

AKS with Nginx and App Gateway  - provides extra hop but better controll over configurations, can use same APP GAteway with multiple AKS Cluster .  can still leverage benefits of WAF,SSL Terminations etc


APIC Error: Permissions on vnet?

az aks show -g rg-aks-demo -n aks-demo-cluster --query "addonProfiles.ingressApplicationGateway.identity.clientId" -o tsv


=== Deploy Sample application and access via App Gateway/ingress===
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml

#kubectl get ingress
it will show 'app gateway' ip address if everything is fine.
