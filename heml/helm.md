# Helm Chart


<img width="341" height="198" alt="image" src="https://github.com/user-attachments/assets/2f174f2f-eef1-4834-a578-a726120b181e" />

```bash

Package manager for Kubernetes, supports templating, versioning, releases.
```



`Helm is an open source tool - it is a package manager for kubernetes. like we have apt , yum in Linux OS`


```bash
Deploy multiple kubernetes objects (Deployments,PODS,Services,ConfigMap,Secrets, etc)  using a single command
```


```bash
It simplifies deployments in Kubernetes - handle/deploy multiple YAML files as a `PACKAGE ` 
```


### Helm allows you to perform:

 install + update + remove package/kubernetes resource
 


### Helm Vs HelmChart ?

- Helm - is a tool/command.
- Helm Chart: deployable package - Combine multiple yaml files ( in a specific structure) as a package basically called helm chart.
- 
`chart is basically a bundle of your application`



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



### Install third-party apps using helm instead of yaml
alot of helm charts already available - you can use that to deploy many applications or kubernetes controllers - like

- Prometheus
- grafana
- nginx
- argoCD

for this purpose . you need to add respective helm repo and then use it..


### OCI Compliant Registry:

- Amazon ECR
- Azure ACR
- GitHub Container Registry



### Helm Chart Structure / Directory Structure:

<img width="1070" height="503" alt="image" src="https://github.com/user-attachments/assets/052e7075-114d-4189-b605-45afa056d003" />


<img width="994" height="372" alt="image" src="https://github.com/user-attachments/assets/53923bf1-0cde-4d5e-bdda-bf563249f716" />


**Optional files:**

<img width="965" height="428" alt="image" src="https://github.com/user-attachments/assets/91fb836d-bb58-431f-8834-1d959c1cc3d8" />


### There are 4 functionalities of a Helm:

**1- Package Manager** (Combine multiple YAML files as a single deployable package)  

**2- Template Engine** (Deploy/Create multiple Micro Services using common/single template file , Useful in CI/CD - pass different values for each micro service)

**3- Multiple Environment Cloning/Replication:** deploy/Run Same application across different environments/clusters/namespaces - change values as per environment

**4- Release Management - (Limited to Helm V2 Only)** - Tiller Server manage release history, creates revision and provide quick rollback features. ( but its not avilable in Helm V3)



# injecting Values in a template:


### How to override default values ( default values we define in values.yaml)

There are a couple of different ways to override  default values:

1- Using -- file=other=values.yaml

2- Using --Set flag



<img width="974" height="512" alt="image" src="https://github.com/user-attachments/assets/42a04f8d-b1bf-4831-97b0-b11afd5b0d67" />


**Final Result:**

<img width="1000" height="316" alt="image" src="https://github.com/user-attachments/assets/b2b37cb7-a542-49bf-a24f-79205445e590" />



Using Commmand line:

<img width="898" height="124" alt="image" src="https://github.com/user-attachments/assets/fdf8973f-72c7-4cda-9454-1b8fee5795c7" />



🔹 **Helm Built-in Objects**

Helm provides several built-in objects you can use in templates:

.Release.Name → The name you passed in helm install.

.Release.Namespace → Namespace where the chart is being installed.

.Release.Revision → Revision number (increments with each upgrade).

.Chart.Name → Name field from Chart.yaml.

.Values → User-defined values from values.yaml or --set.

✅ So, .Release.Name is retrieved automatically from the Helm CLI command at install/upgrade time — it’s not in values.yaml, but injected by Helm itself as a built-in object.


**Reference**:
https://www.youtube.com/watch?v=-ykwb1d0DXU
