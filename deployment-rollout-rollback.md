# Kubernetes Deployments and Strategies used to do Rollout and Rollback

`Rolling updates and Rollouts are the major benefits/features of a deployment object in Kubernetes.`

##  Rolling updates / RollOut / Rollback


**RollOut**:???  `The process of deploying new versions`    \
      ---> **Rolling Update** - it is a deployment/rollout strategy to deploy a new versoin - updates your application one pod at a time, without any downtime. It replaces the old version with the new one gradually.




**Rollback**  --> “That last update broke things. Please go back to the previous working version.” \
`It’s like an undo for deployments.`  \
                    `Revert to the last stable version`






### Deploy or Update a Deployment or change its image/version :

```bash
kubectl set image deployment/my-app nginx=nginx:1.260
kubectl set image deployment/my-deployment my-container=my-image:v2
```
