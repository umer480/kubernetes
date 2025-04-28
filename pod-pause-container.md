### PAUSE Container ( aks infra container)
special hidden container: Think of it like a silent manager inside the Pod that keeps the Pod's networking running while app containers come and go.

POD is nothing but a logical boundary for containers, one we create a pod then by default it has 1 default container which is called ‘**PAUSE**” container and pause container holds networking information/ip address for the entire pod _ means all container with the same pod use the same ip address that is associated or manage by pause container and all containers act as a LOCALHOST.

When you try to deploy a container or multiple containers in a Pod, Kubernetes **creates a Pause container** in the Pod.
In Kubernetes, the "pause" or "infrastructure" container is a lightweight container that acts as a parent, ensuring the pod's networking and namespace setup persists even if other containers crash or restart.


## POD internal structure:
![image](https://github.com/user-attachments/assets/5f22294e-f6df-4b1a-af94-99b9850559c7)


## **How least many containers a pod can have:**
So every pod has atleast 2 containers ( one is Pause and one custom to run our actual application)


**We can list /see the pause containers using command :**
```bash
docker container ls
docker ps | grep pause

```

## **NOTE:** POD will not run if we don’t specify a custom container – means a pod cannot run with only a pause container
If the pause container crashes → your whole Pod is broken.

## All containers join the same pause container (means join the same networking)
if any of the container restart or crash then it join the same pause network **(means uses the same ip address that previously it was using) - this makes consistency and provide stable ip / network connectivity to containers that resides within the same pod.**


![image](https://github.com/user-attachments/assets/a099167c-25c3-4ca1-b40d-60008722b278)
