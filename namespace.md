### NameSpace in Kubernetes

## Namespace: A virtual cluster. - Logical grouping of kubernetes resources
Namespaces allow Kubernetes to manage multiple clusters (for multiple teams or projects) within the same physical cluster.

**Namespace** = a way to divide and organize Kubernetes resources logically inside the same cluster.


**This is very useful when:**
You have multiple teams using the same cluster.
You want to separate environments like dev, test, and prod or for multiple projects.
You want to limit resources (like CPU, memory) for different groups.


![image](https://github.com/user-attachments/assets/9e06e0d3-5d5b-4168-afe1-24a773b01b1a)

![image](https://github.com/user-attachments/assets/c4c41a29-a8eb-421d-9537-05cf8d075d51)


## Default Namespaces in Kubernetes
**default** → Where stuff goes if you don’t specify a namespace.

**kube-system** → For Kubernetes internal system components


## Commands
```bash
kubectl get namespaces
kubectl create namespace <namespace-name>
kubectl config set-context --current --namespace=<namespace-name>    -->Set a default namespace in your kubeconfig
kubectl describe namespace <namespace-name>
kubectl describe namespace <namespace-name>
```



**Example **:
kubectl create namespace dev
kubectl get pods -n dev
kubectl delete namespace dev
```

## Create a namespace using manifest/yml file:
```bash
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "name": "production",
        "labels": {
            "name": "production"
        }
    }
}
```


## Can different namespace communicate with each other?
**Yes,** different namespaces can communicate with each other **by default**! (Unless you specifically block them.)

BUT !!!
**If you want to restrict communication, you can:**
- Use Network Policies to block or allow traffic between namespaces.
- Set up RBAC (Role-Based Access Control) to limit who can access what.



**How POD Communicate happen between difference namecpace:**
Pods in one namespace can talk to pods in another namespace using the full DNS name.
#Uses full DNS name including the namespace.

The DNS name looks like:
```bash
<service-name>.<namespace-name>.svc.cluster.local
```
A Service called frontend in the dev namespace,
A pod in the test namespace can reach it by calling:

```bash
http://frontend.dev.svc.cluster.local
```

```bash
+----------------------+            +-----------------------+
|  Namespace: test     |            |  Namespace: dev       |
|                      |            |                       |
|  +----------------+  |            |  +----------------+   |
|  | Pod: test-pod   | |  ----->    |  | Service: frontend| |
|  +----------------+  |   (DNS)    |  +----------------+   |
|                      |            |      ↓                |
+----------------------+            |  +----------------+   |
                                    |  | Pod: frontend-pod| |
                                    |  +----------------+   |
                                     +-----------------------+

```


### Key Points : Take aways

🔹 By default, all pods in a cluster can communicate, even if they are in different namespaces.

🔹 Kubernetes does not create strict network isolation between namespaces unless you manually configure it using Network Policies.

🔹 Services in different namespaces can be accessed using the fully qualified domain name (FQDN)
