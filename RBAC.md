# RBAC in kubernetes

`RBAC (Role-Based Access Control) in Kubernetes is a security mechanism that controls who can do what in your cluster`

### RBAC allows you to define:

`Which users (or service accounts) can perform which actions on which resources`

**It enforces access rules on**:

- API resources (pods, deployments, secrets, etc.)

- Verbs (get, list, create, delete, etc.)

- Namespaces (to scope access)

- 

| Purpose                          | Example                                                 |
| -------------------------------- | ------------------------------------------------------- |
| 🔒 **Restricting access**        | Developers can only `get`/`list` pods, not delete them  |
| 👨‍💻 **Delegating admin roles** | Give team leads admin access to specific namespaces        |
| 🤖 **Securing automation**       | Allow CI/CD pipelines (service accounts) to deploy apps |
| 🕵️‍♂️ **Auditing & compliance** | Enforce least-privilege access by role                      |


### RBAC Key Concepts:


| Object               | Description                                                              |
| -------------------- | ------------------------------------------------------------------------ |
| `Role`               | Defines what actions are allowed on which resources (within a namespace) |
| `ClusterRole`        | Like Role, but for **cluster-wide** resources                            |
| `RoleBinding`        | Grants a Role to a **user/service account** in a namespace               |
| `ClusterRoleBinding` | Grants a ClusterRole **cluster-wide** or in multiple namespaces          |



Whenever API server receive request on kubernetes cluster then it first check from which account/user request is received. and perform authenticatoin then perform authorization.


![image](https://github.com/user-attachments/assets/888f56e3-06d6-4c99-bef8-45db6edae929)

### Multiple Ways to access/authenticate K8s Cluster:

1- Static Password file --File based approach -  ( this files contains all users/passwords)
2- Static Token File --File based approach
3- SSL Certificate based authentication ( user present SSL and API Server aprove/deny access based on it)

![image](https://github.com/user-attachments/assets/6c3e3311-ba60-41aa-af70-bbe21bd3be3a)

![image](https://github.com/user-attachments/assets/6b6e0bdc-22e9-4be7-b250-cfa329e62cfb)


![image](https://github.com/user-attachments/assets/71517376-5b22-4e87-9eba-6d041beafa29)


![image](https://github.com/user-attachments/assets/1c7c1c97-cbde-4b09-baf1-3ca15d03f721)






### Authorization

![image](https://github.com/user-attachments/assets/4e37979c-2a6c-4b78-bd02-ac09167a694e)


### Authorization Modes:

![image](https://github.com/user-attachments/assets/fe1ddcac-1465-43ea-8030-68e39cd4eed6)


**1** - Attribute-based access control. in which we create a policy abject and assing it to a single user

![image](https://github.com/user-attachments/assets/1303b198-df60-48ac-b1ae-b6b26c103a17)

**2** RBAC (Role-Based Access Control)

- A Role contains certain permissions.  
- This is useful when you want multiple users to have the same level of permissions/access.
- Single Role can assing to multiple users ( using RoleBinding)

![image](https://github.com/user-attachments/assets/681dbd00-f6b9-4fcc-bad1-b08d70f134e4)



### More Authorization Modes:

![image](https://github.com/user-attachments/assets/46a4eff1-1242-47d7-8d8f-f23246834451)



### Objects in Role-Based Access Control (RBAC)

![image](https://github.com/user-attachments/assets/04134e7a-c88f-4580-ac3f-8b1b7613ec80)



### ROLE:

 Role is a NameSpaced object - means resources that you deploy within a namespace can be accessed/managed via ROLE.
 

 ![image](https://github.com/user-attachments/assets/9a10af23-960f-4ed8-8cd1-a1ba629960ee)

The role.yaml file contains the configuration for creating a role named pod-reader. The role allows the user to perform actions like get, watch, and list on pods resources.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

```

```bash
kubectl get role
```



### Role Binding:
Role binding is a way to attach a role ( or assing)  with a user

![image](https://github.com/user-attachments/assets/754f5543-ec81-4933-9884-4c47bffe77b3)

![image](https://github.com/user-attachments/assets/0d14d1de-e334-4ad1-9070-e5fbf817aaeb)

```bash

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jack
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

```

```bash
kubectl get rolebinding
```

To check the permissions of the jack user:

```bash
kubectl auth can-i get pod --as jack
```



|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
## ROLE vs ClusterRole

🔹 **Role**
**Scope**: Namespace-scoped   **<---**

Usage: Grants access within a specific namespace.

Can manage: Resources like pods, services, configmaps within that namespace only.

Created with: kind: Role

🔹 **ClusterRole**
**Scope**: Cluster-wide    **<---**

Usage: Grants access to:

Cluster-scoped resources (e.g., nodes, persistent volumes)

Namespaced resources across all namespaces

Created with: kind: ClusterRole

|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||

### Cluster Role

Gain access on cluster Wide resources (nodes,namespaces,pv etc)

   'it can also use to gain access on all namecpases, pod of all namespaces' 

![image](https://github.com/user-attachments/assets/75e70a15-d334-44ed-9778-dcfa980ff18c)


The clusterrole.yaml file contains the configuration for creating a cluster role named secret-reader. This cluster role allows the user to perform actions like get, watch, and list on secrets resources.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]

```

```bash
kubectl get clusterrole
```


### Cluster Role Binding

![image](https://github.com/user-attachments/assets/0af1242b-bea4-48eb-ab16-18d236e8ce92)

The clusterrolebinding.yaml file contains the configuration for creating a cluster role binding named read-secrets-global. This cluster role binding binds the secret-reader cluster role to the user riya globally.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: User
  name: riya
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io

```

```bash
kubectl get clusterrolebinding
```

To check the permissions of the riya user across all namespaces:

```bash
kubectl auth can-i get secret --as riya -A
```

### Role Binding (Namespace-level)
The rolebinding.yaml file defines a role binding named read-secrets that binds the secret-reader cluster role to the user dev in the development namespace.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  namespace: development
subjects:
- kind: User
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io

```

```bash
kubectl get rolebinding
```


To check the permissions of the dev user in the development namespace:

```bash
kubectl auth can-i get secret --as dev -n development
```


## LAB

1- Create a user identity (for example, via client certificate or an external identity like OIDC).

2- Create a Role or ClusterRole defining what the user is allowed to do.

3- Bind the Role to the user using a RoleBinding or ClusterRoleBinding

4-  Kubeconfig context for the new user


### Configure kubeconfig for Jane


```bash

kubectl config set-credentials jane \
  --client-certificate=jane.crt \
  --client-key=jane.key \
  --embed-certs=true

kubectl config set-context jane-context \
  --cluster=<your-cluster-name> \
  --user=jane \
  --namespace=dev

kubectl config use-context jane-context
```

Replace <your-cluster-name> with your actual cluster name (check with kubectl config get-clusters).

Now you can test:

```bash
kubectl get pods   # Allowed
kubectl delete pods  # Should be denied
```



### Create a Service Account
`Use ServiceAccount Instead of Certificate User`

ServiceAccounts are often easier and recommended for automation.

```bash
kubectl create serviceaccount rbac-user -n dev

kubectl create rolebinding rbac-user-binding \
  --role=pod-reader \
  --serviceaccount=dev:rbac-user \
  --namespace=dev

```


Then you can use the ServiceAccount token from a pod or export it for CI/CD.






https://www.youtube.com/watch?v=MGCF6slXG0w
https://github.com/devopsproin/certified-kubernetes-administrator/tree/main/RBAC


