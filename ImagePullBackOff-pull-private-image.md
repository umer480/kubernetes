
# What is ImagePullBackOff  - Troubleshooting
`
ImagePullBackOff` is a Kubernetes Pod status that indicates Kubernetes is unable to pull the container image from the specified image repository. It goes into a "back-off" state, which means it tries to pull the image, fails, and then waits for a bit before retrying — increasing the wait time gradually


# **Troubleshooting ImagePullBackOff Error:**

![image](https://github.com/user-attachments/assets/914ca2cc-05a1-47c3-a72e-239f5c4cb1b8)



# Reasons of imagePullBackOff:

- incorrect image name/ incorrect tag
- image does not exist(owner could remove)
- Network connectivity issue (worker node unable to reach destination / on that server image resides)
- private image ( incorrect/expired credentials)
- Rate limiting  (rate limits for unauthenticated pulls)




# How to pull an image from a Private Registry in K8s

**Private Registry :**

- Nexus
- Docker HUB
- AWS ECR
- Azure ACR
- -Others...



# Steps:

1 - Create a Secret ( that contains access token/credentials to access your private registry)

2 - Configure Deployment/POD ( use secret using 'imagePullSecrets' ) 


![image](https://github.com/user-attachments/assets/359a4412-8d21-4c31-8ca8-e34f940f672a)






**Config file path :** 
.docker/config.json

this file created automatically in user's home directory when you login via docker `#docker login -u umerazeem https://index.docker.io/v1/`




# Create a Secret : 

`Dry run:`

```bash
kubectl create secret docker-registry dockerhubcred --docker-server=https://index.docker.io/v1/  --docker-username=umerazeem --docker-password=dckr_pat_PKkO3WCAxScuYlo0UcY6gD-EKsgxx --dry-run=client -o yaml
```

`Create`:
```bash
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/  --docker-username=umerazeem --docker-password=dckr_pat_PKkO3WCAxScuYlo0UcY6gD-EKsgxx
```

# Create a deployment that can access private image :

use :  `imagePullSecrets`


```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: privateimage-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      imagePullSecrets:                   # <--- Added this section
        - name: dockerhubcred                   # <--- Replace with your secret name
      containers:
        - name: nginx-container
          image: umerazeem/cloudlyncs:v1
          ports:
            - containerPort: 80

```

# Question !!!
What if image is locally available on node but removed from backend on server? do it download again?



# If Image is Already Available on the Node:
`If the image is already present on the node, Kubernetes does not pull it again unless you specify the imagePullPolicy.`

**More explanation about default behaviour:**
The default behavior is:

- `Always` if the tag is latest.

- `IfNotPresent` if a specific version is specified.

- `Never` if you explicitly set it to Never.



```bash

spec:
  containers:
    - name: my-container
      image: umerazeem/my-private-image:1.0.0
      imagePullPolicy: IfNotPresent  # Will only pull if the image is not available locally

```



**If you want to force pull even if the image is present:**

```bash
spec:
  containers:
    - name: my-container
      image: umerazeem/my-private-image:1.0.0
      imagePullPolicy: Always  # Always pulls the image from the registry
```



