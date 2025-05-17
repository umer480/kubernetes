# init Containers

These are the normal containers except they do some tasks before the main containers and always exits, this means they are not long-running containers but the containers have some specialized tasks. They come to do the task and complete after that the main containers start to come up.

![image](https://github.com/user-attachments/assets/362f656b-5831-44c4-9f72-94284f9e8361)


'A single pod can have multiple init containers. They can run one by one (sequentially) and complete their tasks.'


# Init-Container Flow:

![image](https://github.com/user-attachments/assets/971ce6df-1446-4376-80fe-8fa0a1734565)



# What are they used for?

**Dependency Management:** Downloading or installing dependencies that are not part of the application container image. This can be useful for applications that require specific data to be present before they can run.

**Configuration:** Fetching and configuring secrets, setting up environment variables, or creating directories.  (An application might need to fetch a secret from a secure management service, which could be done by an init container before the application container start)

**Security Concern:**

These containers are used to prepare the pod to run the main application containers. These tasks can be changing permissions of some files to changing some specific environments. They can also be used as a precheck to verify if the application can be run on this pod.
There may be some tasks that only the root user can do. And since you never want to run your application as a root user as this is a security threat. Init containers can do the task for you that you wanted to run as a root user.

**Run database tasks:**
An init container can run database migrations or schema updates before the main application container starts. This ensures that the application has access to the latest database schema.

Init Containers run sequentially and must complete successfully before the main application starts.

# Creating init Container:

To use init containers, you simply need to add them to the pod specification in your Kubernetes deployment manifest. Let’s see it in action.

```bash

apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  initContainers:
  - name: init-container # Init Container.
    image: busybox
    command: ["/bin/sh", "-c", "echo 'Hello I am Init Container';", "sleep 60"]
  containers:
  - name: app-container  # Application Container
    image: nginx:latest

```

The above deployment YAML has the settings for both containers—the init container and the app container. Executing this deployment file will initiate the following sequence of events:


```bash
init-container (Running) → app-container (Init Phase) → init-container (Completed) → app-container (Running)
```


# Check POD Status and init Container Logs:

![image](https://github.com/user-attachments/assets/26adec2a-d056-4996-8588-f3df27311b2d)


# Customize behavior of init container as per your application requirements and needs:

An init container can execute custom scripts or commands to perform any necessary initialization tasks before the main application container start.





# BusyBox image:
BusyBox is a lightweight image that bundles essential linux utilities ( ping, nslookup,dig,netstat, etc.) that are useful for troubleshooting.
Busybox also useful for init contianer and side cars as a base image.

Note: Busy box does not has a bash shell by default, you have to use sh shell. while using #kubectl exec command with it.




# Lab 1 - First real-world example.

The below example shows the deployment of an application that heavily depends on a database. So until the database doesn’t come into a running state, the application should wait.

```bash
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
    spec:
      containers:
        - name: db-container
          image: redis:6.2.6
          ports:
            - containerPort: 6379 

---
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: db
  ports:
    - port: 6379
      targetPort: 6379 # Please note this port.

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
    spec:
      initContainers:
        - name: init-container
          image: busybox:1.35
          command: ['sh', '-c', 'until nc -w 2 -z db-service 6379; do echo waiting for db; sleep 2; done;']
      containers:
        - name: app-container
          image: nginx:1.21.6
          ports:
            - containerPort: 80
```

The init container within the application deployment continuously monitors the status of the database pod using the Kubernetes database service db-service.

```bash
until nc -w 2 -z db-service 6379; do echo waiting for db; sleep 2; done
```


# LAB : 
LAB : don’t add content in actual image, get using init container in the mounted volume and then consume from the actual container of the app
https://github.com/KamranAzeem/kubernetes-katas/blob/master/04-init-and-multi-container-pods.md


LAB for init container: Real scenario based examples
https://medium.com/@dinesh.pundkar/init-peace-init-containers-17f448264c78



