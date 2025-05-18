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
    command: ["/bin/sh", "-c", "echo 'Hello I am Init Container'; sleep 60"]
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

apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:            # Missing template section added
    metadata:
      labels:
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
      targetPort: 6379
  type: ClusterIP        # Explicitly added (default type), can be changed if needed

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
  template:              # Missing template section added
    metadata:
      labels:
        app: nginx
    spec:
      initContainers:
        - name: init-container
          image: busybox:1.35
          command: ['sh', '-c', 'until nc -vz db-service 6379; do echo "Waiting for Redis DB..."; sleep 30; done; echo "Redis DB is now reachable!"']

      containers:
        - name: app-container
          image: nginx:1.21.6
          ports:
            - containerPort: 80

```


The init container within the application deployment continuously monitors the status of the database pod using the Kubernetes database service db-service.

```bash
['sh', '-c', 'until nc -vz db-service 6379; do echo "Waiting for Redis DB..."; sleep 30; done; echo "Redis DB is now reachable!"']
```


# LAB :  prepare app content (static web content) using init container and make it available for main app ( via shared volume)


```bash

apiVersion: v1
kind: Pod
metadata:
  name: init-container-demo
  labels:
    app: web-content
spec:
  initContainers:
  - name: helper
    image: alpine/git
    command:
    - git 
    - clone
    - https://github.com/umer480/devops-sample-web.git
    - /web-content/
    volumeMounts:
    - name: web-content-dir
      mountPath: "/web-content"
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: web-content-dir
      mountPath: /usr/share/nginx/html
  volumes:
  - name: web-content-dir
    emptyDir: {}

```

Expose Service for testing:

```bash
kubectl expose pod init-container-demo --type=NodePort
```

# Question !  **How downlaoded files are accessible for main app container:**
multiple containers within the same POD:
They only share the network namespace but not the filesystem or process space.


If you want an Init Container to "share" something with the main container, you have to use a shared volume. This is the main mechanism for communication between Init Containers and main containers.

**shared volumes:**

Create a shared volume.

Mount that volume in both Init and main containers.

The Init Container installs or downloads the software/tools into the shared volume.

The main application accesses those tools or files from the shared volume.

