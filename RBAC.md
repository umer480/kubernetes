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



https://www.youtube.com/watch?v=MGCF6slXG0w
https://github.com/devopsproin/certified-kubernetes-administrator/tree/main/RBAC


