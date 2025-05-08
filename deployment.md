Label:


![image](https://github.com/user-attachments/assets/66d41c9c-87bd-4947-a445-0def21d5514a)


All pods of same deployment gets different unique name.However,
We can identify all the pods replicas of the same application using a specific label that all of them share
Template (in yml)  define properties of pod
So in template ‘we define label for a pods, which is mandatory property to identify it



4. Scale the Deployment Down and Up:
You can scale down the number of replicas to 0 and then scale it back up to the desired number of replicas. This will also recreate the pods.
bash
Copy code
kubectl scale deployment <deployment-name> --replicas=0
kubectl scale deployment <deployment-name> --replicas=<desired-number>
Choose the method that best suits your situation based on how your pods are managed.



POD automatic creation:

If you deployed a deployment – and even set replica=1 and manually delete the container, the deployment will automatically create the container – make it healthy – this is called self healing.



### Standalone POD issues/limitations:
While you can create individual Pods directly, they are:

**Not resilient:** If a Pod crashes, it will not restart automatically.

**Not scalable:** You need to manually create multiple Pods.

**Not version controlled**: Updates are manual, and rolling back is tedious.


### What is Deployment: 
A Deployment in Kubernetes is a higher-level abstraction that manages a set of replicated Pods. It provides a way to:

kind : deployment


**Deployments solve this by:**

**Self Healing :** Maintaining the desired number of replicas

**Rolling updates :** Allowing seamless updates (.

**Rollback :** Enabling easy rollback to previous versions.

**Consitency:** Ensuring consistent configuration across multiple Pods.


![image](https://github.com/user-attachments/assets/675fd057-5428-4170-b06e-1daa1ad1954c)




Sample YAML file for deployment:

```bash
apiVersion: apps/v1                # API version for the Deployment resource
kind: Deployment                   # Defines this resource as a Deployment
metadata:
  name: nginx-deployment           # Name of the Deployment
  labels:
    app: nginx                     # Labels used to identify the deployment
spec:
  replicas: 3                      # Number of pod replicas to maintain
  selector:                        # Selector to match pods with the correct labels
    matchLabels:
      app: nginx                   # This must match the pod template's labels
  template:                        # Defines the pod template for this deployment
    metadata:
      labels:
        app: nginx                 # Labels for the pods created by this template
    spec:
      containers:                  # List of containers within the pod
        - name: nginx-container    # Name of the container
          image: nginx:1.21        # Docker image to use for this container
          ports:
            - containerPort: 80    # Expose port 80 from the container


```

**To list the pods created by the deployment:**
```bash
kubectl get pods -l app=nginx
```



**Expose a Deployment via Service and access outside Cluster:**

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service               # Name of the Service
  labels:
    app: nginx                      # Labels to identify the service
spec:
  selector:
    app: nginx                      # Matches the label of the Deployment pods
  type: LoadBalancer                # Type of service (LoadBalancer for external access)
  ports:
    - protocol: TCP
      port: 80                      # Exposed port of the service
      targetPort: 80                # Port the container listens on

```

Using nodeport instead of Loadbalancer:

```bash
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080    # This will open port 30080 on your node

```

**When you apply this:**

- A Deployment is created.

- It creates a ReplicaSet automatically.

- The ReplicaSet maintains 3 replicas of the specified Pod.


### Commands
kubectl get deployments	                            List all deployments in the namespace.
kubectl describe deployment <name>	                Show detailed information about a deployment.
kubectl delete deployment <name>	                  Delete a specific deployment.
kubectl scale deployment <name> --replicas=5	      Scale the deployment to 5 replicas.
kubectl rollout status deployment <name>	          Check the status of the latest rollout.
kubectl rollout undo deployment <name>	            Rollback to the previous deployment version.
kubectl edit deployment <name>	                    Edit the deployment configuration live.
kubectl rollout restart deployment <name>	          Restart all pods in the deployment.



🔹 Relationship Between Deployment, ReplicaSet, and Controller
In Kubernetes, Deployments, ReplicaSets, and Controllers work together to manage application workloads in a scalable and fault-tolerant way.
deployment automatically creates and manages ReplicaSets.

Rolling Updates and Rollback are not available in ReplicaSet Object

**🔹 ReplicaSet YAML Example (Not Recommended Directly)**

If you use a ReplicaSet directly, it will maintain 3 Pods, but without update or rollback features.

This is why Deployments are recommended.


```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3                          # Number of replicas
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx:1.21

```


 **Key Notes :**

**ReplicaSets** — Maintaining multiple copies of a Pod.

**Deployments**— Managing updates, rollbacks, and scaling

