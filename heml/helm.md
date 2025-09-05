# Helm Chart


<img width="341" height="198" alt="image" src="https://github.com/user-attachments/assets/2f174f2f-eef1-4834-a578-a726120b181e" />


`Helm is an open source tool - it is a package manager for kubernetes. like we have apt , yum in Linux OS`


```bash
Deploy multiple kubernetes objects (Deployments,PODS,Services,ConfigMap,Secrets, etc)  using a single command
```


```bash
It simplifies deployments in Kubernetes - handle/deploy multiple YAML files as a `PACKAGE ` 
```


### Helm Vs HelmChart ?

- Helm - is a tool/command.
- Helm Chart: deployable package - Combine multiple yaml files ( in a specific structure) as a package basically called helm chart.




### Helm Hub / Artifacts Hub

```bash
https://artifacthub.io/
```

### Search helm packages

- Via UI
- Via Command


```bash
helm search <package>
```

### Download Helm Package locally

```
helm pull --untar oci://registry-1.docker.io/bitnamicharts/mysql --version 14.0.3
```

### WorkFlow

<img width="1512" height="905" alt="image" src="https://github.com/user-attachments/assets/b04b37e7-dcef-44b7-a290-c2928cf00107" />



### OCI Compliant Registry:

- Amazon ECR
- Azure ACR
- GitHub Container Registry



### Helm Chart Structure / Directory Structure:



### There are 4 functionalities of a Helm:

**1- Package Manager** (Combine multiple YAML files as a single deployable package)  

**2- Template Engine** (Deploy/Create multiple Micro Services using common/single template file , Useful in CI/CD - pass different values for each micro service)

**3- Multiple Environment Cloning/Replication:** deploy/Run Same application across different environments/clusters/namespaces - change values as per environment

**4- Release Management - (Limited to Helm V2 Only)** - Tiller Server manage release history, creates revision and provide quick rollback features. ( but its not avilable in Helm V3)



# injecting Values in a template:


### Overwrite Values:

1- Using -- file=other=values.yaml

2- Using --Set flag
