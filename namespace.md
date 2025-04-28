### NameSpace in Kubernetes

## Namespace: A virtual cluster. - Logical grouping of kubernetes resources
Namespaces allow Kubernetes to manage multiple clusters (for multiple teams or projects) within the same physical cluster.

![image](https://github.com/user-attachments/assets/9e06e0d3-5d5b-4168-afe1-24a773b01b1a)

```bash
![image](https://github.com/user-attachments/assets/75d98d77-690d-4164-9f39-ef5807c819be)
```


## Create a namespace using manifest file:
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


🔹 By default, all pods in a cluster can communicate, even if they are in different namespaces.
🔹 Kubernetes does not create strict network isolation between namespaces unless you manually configure it using Network Policies.
🔹 Services in different namespaces can be accessed using the fully qualified domain name (FQDN)


