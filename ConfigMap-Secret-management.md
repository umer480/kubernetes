

### How to add or update existing config and secret in already running deployments/pods ? 


**Modify Existing Objects:**
#kubectl edit command  -> Manual  Quick change/dev/test  (bypassing CI/CD)
#Using yml file   -> Recommended



1- **Volume based:** it will reflect automatically within 60 sec by kubelet
2- **ENV variable based**: #kubectl rollout restart deployment <deployment name>

**validation :** #kubectl exec -it <pod-name> -- /bin/bash   >  # env

its also mandatory to resrat pods if your application cache the configs/secret in memory.


**rollout restart** does not delete or recreate the deployment itself but initiates a rolling restart of all its pods.
means, It gradually creates new pods while terminating the old ones.

**Useful for:**
- Fetch updated ConfigMaps or Secrets.
- New image versions (if the image was updated externally).



# 🚀 Key Points:

**Safe Restart**: It uses Kubernetes' rolling update mechanism—no downtime.
**No Configuration Change:** It only restarts the pods, not the deployment definition.
**Quick Recovery:** Useful for refreshing pod states without redeploying.


### Adding new Secret/ ENV Variable in deployment:

#kubectl rollout restart  command does not modify any configuration (like environment variables, image version) -
What if you want to add new env variable in deployment? you have to apply yml that containes new env in specs or edit live yml using kubectl edit - it will restart/rollout deploymnet 


