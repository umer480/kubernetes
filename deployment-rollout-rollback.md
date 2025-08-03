# Kubernetes Deployments and Strategies used to do Rollout and Rollback

`Rolling updates and Rollouts are the major benefits/features of a deployment object in Kubernetes.`

**RECREATE Strategy** :
Terminate all old pods before creating new ones.




##  Rolling updates / RollOut / Rollback


**RollOut**:???  `The process of deploying new versions`    \
      ---> **Rolling Update** - it is a deployment/rollout strategy to deploy a new versoin - updates your application one pod at a time, without any downtime. It replaces the old version with the new one gradually.




**Rollback**  --> “That last update broke things. Please go back to the previous working version.” \
`It’s like an undo for deployments.`  \
                    `Revert to the last stable version`


### There are below 3 deployment strategies that we usually follow for rollout in the organizations:

1- **Rolling Update** : 
Some clients may connect to the old version, and some to the new version — at the same time — until the update is complete.



2- **Blue-Green Strategy**:
creates a new environment and switch traffic all at once from old to new.

3- **Canary deployment**:
Send a small percentage of traffic  (a subset of users) to the new version before rolling it out fully.





### Rolling updates  - tehcnical deep drivre and implementation:

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

**Step **3: Repeat the process

**Now**:

again it deletes 1 more old pod → 2 old pods remain

Creates 1 more new pod → 2 old + 2 new

--- continue and so on ....





**Other Scenarios**:

`     --> replicas =4 ,   MaxSurge= 0 , Maxunavailable= 4 `

 `    --> replicas =4   , MaxSurge = 2 , Maxunavailable = 0 `






### Deploy or Update a Deployment or change its image/version using a command:

```bash
kubectl set image deployment/my-app nginx=nginx:1.260
kubectl set image deployment/my-deployment my-container=my-image:v2
```
