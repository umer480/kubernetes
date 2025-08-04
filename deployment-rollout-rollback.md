# Kubernetes Deployments and Strategies used to do Rollout and Rollback

`Rolling updates and Rollouts are the major benefits/features of a deployment object in Kubernetes.`



##  Rolling updates / RollOut / Rollback


**RollOut**:???  `The process of deploying new versions`    \
      ---> **Rolling Update** - it is a deployment/rollout strategy to deploy a new versoin - updates your application one pod at a time, without any downtime. It replaces the old version with the new one gradually.




**Rollback**  --> “That last update broke things. Please go back to the previous working version.” \
`It’s like an undo for deployments.`  \
                    `Revert to the last stable version`


### There are below 4 deployment strategies that we usually follow for rollout in the organizations:


1- **Recreate** : Terminate all old pods before creating new ones.

2- **Rolling Update** : 
Some clients may connect to the old version, and some to the new version — at the same time — until the update is complete.

3- **Blue-Green Strategy**:
creates a new environment and switch traffic all at once from old to new.

4- **Canary deployment**:
Send a small percentage of traffic  (a subset of users) to the new version before rolling it out fully.





### Rolling updates  - technical deep dive and implementation:

In Kubernetes, when you perform a rolling update (which is the default deployment strategy), you can control how many pods are updated at a time using two key settings in your Deployment spec:

⚙️ **Key Fields to Control Rolling Updates**

| Parameter        | What it does                                                                                       |
| ---------------- | -------------------------------------------------------------------------------------------------- |
| `maxSurge`       | How many **extra pods** (new version) can be created **above desired replicas** during the update. |
| `maxUnavailable` | How many **existing pods** (old version) can be **taken offline** during the update.               |



### Example: ✅ ### Default Deployment Strategy in Kubernetes:

When you do not define maxSurge and maxUnavailable, **Kubernetes uses**:

```bash

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```

### Example:  🔢 You have 4 replicas in the Deployment:

Let’s understand the values:

**maxSurge**:         25% of 4 = 1 → 1 extra pod allowed during rollout

**maxUnavailable**:   25% of 4 = 1 → 1 pod can be down at a time



### 🔁 Step-by-step Rolling Update:

**Step 0**: Initial state
4 pods running with old version (v1)

**Step 1**: Kubernetes starts the rollout
It can take down up to 1 old pod (maxUnavailable = 1)

It takes down 1 old pod → now 3 old pods remain

It creates 1 new pod (maxSurge = 1) → now 4 pods total (3 old + 1 new)

**Step 2**: Wait for the new pod to become Ready
Once the new pod is healthy, Kubernetes proceeds.

**Step 3:**   Repeat the process

**Now**:

again it deletes 1 more old pod → 2 old pods remain

Creates 1 more new pod → 2 old + 2 new

--- continue and so on ....





**Other Scenarios**:

`     --> replicas =4 ,   MaxSurge= 0 , Maxunavailable= 4 `

 `    --> replicas =4   , MaxSurge = 2 , Maxunavailable = 0 `
 

`With maxUnavailable: 0, old pods are not removed until new ones are ready.`

`maxSurge: 100%:` up to double the number of pods can temporarily exist.

**Resource pressure - if maxSurge >>>> / 100%** 

'Adding surge pods might consume CPU/memory, causing old pods to slow down or fail probes, causing them to go NotReady.'

More pods = more CPU/memory usage. This can overload nodes if you don’t have room.

Cost impact: On autoscaling clusters, a high surge may trigger extra node provisioning.


### Deploy or Update a Deployment or change its image/version using a command:

```bash
kubectl set image deployment/my-app nginx=nginx:1.260
kubectl set image deployment/my-deployment my-container=my-image:v2
```




# Rollback   -undo deployment changes: restore to previous version

In Kubernetes, a rollback is the process of reverting a Deployment back to a previous stable version if the latest rollout fails or causes issues (like crashes, downtime, or bugs).


**You might want to rollback if**:

The new pods crash or fail health checks.

You accidentally deployed the wrong version.

Users report issues after a new rollout.

Metrics show degraded performance.



**When you trigger a rollback**:

1-It starts a new rollout using that older version.

2-The broken version is replaced safely (similar to a normal rollout).




### Revision number:

`Rollout creates a revision number.`


When you create a deployment, an automatic rollout is triggered, generating a number known as a Revision.

Any modification made to the deployment’s container template/spec will also trigger a rollout, and a new revision is created for each change.

Kubernetes automatically creates a new revision every time you update the deployment spec (e.g., change image, environment variables, etc.).

<img width="373" height="536" alt="image" src="https://github.com/user-attachments/assets/adc87552-bd23-48d5-9b43-29365d80d931" />  \



============================================================




<img width="597" height="434" alt="image" src="https://github.com/user-attachments/assets/ece28f22-188d-4975-9e60-a867b37659e1" />



### How to Set Change Cause for a Revision:

✅ This allows kubectl rollout history to show the "CHANGE-CAUSE" for better audit and understanding.


**Method:1**


Set annotation in deployment metadata;

```bash
  annotations:
    kubernetes.io/change-cause: "Updated image to nginx 1.28 - fixed bug"  # <-- Change cause annotation
```


**Example**:

```bash
apiVersion: apps/v1                # API version for the Deployment resource
kind: Deployment                   # Defines this resource as a Deployment
metadata:
  name: nginx-deployment           # Name of the Deployment
  annotations:
    kubernetes.io/change-cause: "Updated image to nginx 1.28 - fixed bug"  # <-- Change cause annotation
  labels:
    app: nginx                     # Labels used to identify the deployment
spec:
  replicas: 4                      # Number of pod replicas to maintain
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
          image: nginx:1.28      # Docker image to use for this container
          ports:
            - containerPort: 80    # Expose port 80 from the container

```

**Method:2**

use `--record=true` while set image command;

If --record is not used, CHANGE-CAUSE will be <none>.


```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26  --record=true
```

**Method:3**
You can also manually set the change-cause using below command:

```bash
kubectl annotate deployment nginx-deployment kubectl.kubernetes.io/change-cause="Updated image to v2"
```



### Important Commands:

**View rollout history**:

```bash
kubectl rollout history deployment nginx-deployment
```

**Rollback to the previous revision**:

```bash
kubectl rollout undo deployment nginx-deployment
```

**Rollback to a specific revision**:

```bash
kubectl rollout undo deployment nginx-deployment --to-revision=2
```


check rollback status:

```bash
kubectl rollout status deployment <deployment-name>
```

### How to inspect specific revision history

if you dont know what exact image configured or what changed in pod's template for a particular revision history then you can inspect it using below command:

```bash
kubectl rollout history deployment nginx-deployment --revision=1
```

<img width="1369" height="383" alt="image" src="https://github.com/user-attachments/assets/545d468c-94ba-425a-b18e-8ec4f2f5115b" />


## Revision history limit:

By default, deployment maintain last 10 revisions. you can update its value in deployment yml `revisionHistoryLimit: 10`

```bash

spec:
  progressDeadlineSeconds: 600
  replicas: 4
  revisionHistoryLimit: 10
```


  
### Restart deployment

The command  is used to restart the Pods in a deployment without changing the deployment spec.

```bash
kubectl rollout restart deployment <deployment-name>
```




**What this command is does it actually**:

It triggers a rolling restart of all pods managed by the deployment.

It updates the deployment.spec.template.metadata.annotations with a new timestamp.

This change causes Kubernetes to see the Pod template as updated → leading to new Pods being created and old ones terminated.

The deployment revision is incremented.


### When you need to restart a deployment?


| Use Case                      | Description                                                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| ConfigMap or Secret Updated   | If a Pod uses ConfigMaps or Secrets and you want the changes to take effect without editing the deployment spec. |
| Recover from transient errors | If Pods are stuck or misbehaving, a restart can fix them.                                                        |
| No change to image or YAML    | You don’t need to change container image or YAML manually.                                                       |



`NOTE: This does not undo a previous rollout or revert the deployment to an earlier version (that's what kubectl rollout undo is for).`



## 🔍 Notes

Rollbacks follow the same rules as rollouts (maxSurge, maxUnavailable, etc.).

Only works if the Deployment had a previous successful rollout.

You can also manually trigger a rollback by changing the image or other spec back to the older version.




### in-flight requests
