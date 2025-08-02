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



Rolling Update : 
Some clients may connect to the old version, and some to the new version — at the same time — until the update is complete.



Blue-Green Strategy:
creates a new environment and switch traffic all at once from old to new.

Canary deployment:
Send a small percentage of traffic  (subset of users) to the new version before full rollout.



In Kubernetes, when you perform a rolling update (which is the default deployment strategy), you can control how many pods are updated at a time using two key settings in your Deployment spec:

⚙️ Key Fields to Control Rolling Updates

| Parameter        | What it does                                                                                       |
| ---------------- | -------------------------------------------------------------------------------------------------- |
| `maxSurge`       | How many **extra pods** (new version) can be created **above desired replicas** during the update. |
| `maxUnavailable` | How many **existing pods** (old version) can be **taken offline** during the update.               |


📌 What I meant by "replace when ready":
When a new pod (v2) is created to replace an old pod (v1):

Kubernetes starts a new pod with the updated container image.

It waits until the new pod becomes "Ready" (i.e., passes health checks).

Only after that, it will terminate one of the old pods.

This continues until all old pods are replaced by new ones.

✅ Example:
If you have 4 pods and:

yaml
Copy
Edit
maxSurge: 1
maxUnavailable: 0
Then during update:

Kubernetes creates 1 new pod (you now have 5 pods temporarily).

When the new pod is "Ready", it terminates 1 old pod.

You again have 4 pods.

Repeat until all 4 pods are new.

So:
✅ "Ready" new pod ➡️ ✅ Replace an old one.
🚫 Not Ready ➡️ 🚫 Don't remove old pod.




### Deploy or Update a Deployment or change its image/version :

```bash
kubectl set image deployment/my-app nginx=nginx:1.260
kubectl set image deployment/my-deployment my-container=my-image:v2
```
