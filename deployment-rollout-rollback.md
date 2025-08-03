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




# Rollout   -undo deployment changes : restore to previous version

In Kubernetes, a rollback is the process of reverting a Deployment back to a previous stable version if the latest rollout fails or causes issues (like crashes, downtime, or bugs).


**You might want to rollback if**:

The new pods crash or fail health checks.

You accidentally deployed a wrong version.

Users report issues after a new rollout.

Metrics show degraded performance.


### 🔙 How rollback works

Kubernetes automatically keeps a history of previous ReplicaSet revisions for a Deployment.

When you trigger a rollback:

1-Kubernetes finds the last known stable ReplicaSet.

2-It starts a new rollout using that older version.

3-The broken version is replaced safely (similar to a normal rollout).




### Revision number:

When you create a deployment, an automatic rollout is triggered, generating a number known as a Revision.

Any modification made to the deployment’s container template/spec will also trigger a rollout, and a new revision is created for each change.



<img width="373" height="536" alt="image" src="https://github.com/user-attachments/assets/adc87552-bd23-48d5-9b43-29365d80d931" />


**View rollout history**:

```bash
kubectl rollout history deployment my-app
```

**Rollback to the previous revision**:

```bash
kubectl rollout undo deployment my-app
```

**Rollback to a specific revision**:

```bash
kubectl rollout undo deployment my-app --to-revision=2
```

## 🔍 Notes
Rollbacks follow the same rules as rollouts (maxSurge, maxUnavailable, etc.).

Only works if the Deployment had a previous successful rollout.

You can also manually trigger a rollback by changing the image or other spec back to the older version.




### in-flight requests
