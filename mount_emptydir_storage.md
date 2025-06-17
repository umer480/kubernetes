# emptyDir storage

https://www.devopsschool.com/blog/kubernetes-volume-emptydir-explained-with-examples/

### Some uses for an emptyDir are:

scratch space, such as for a disk-based merge sort
checkpointing a long computation for recovery from crashes
holding files that a content-manager container fetches while a webserver container serves the data



## Here are the following facts for emptyDir storage type in Kubernetes

- An emptyDir volume is first created when a Pod is assigned to a Node and initially its empty
- A Volume of type emptyDir that lasts for the life of the Pod, even if the Container terminates and restarts.
- If a container in a Pod crashes the emptyDir content is unaffected.
- All containers in a Pod share use of the emptyDir volume .
- Each container can independently mount the emptyDir at the same / or different path.
- Using emptyDir, The Kubelet will create the directory in the container, but not mount any storage.
- Containers in the Pod can all read/write the same files in the emptyDir volume, though that volume can be mounted at the same or different paths in each Container.
- When a Pod is removed from a node for any reason, the data in the emptyDir is deleted forever along with the container.
- A Container crashing does NOT remove a Pod from a node, so the data in an emptyDir volume is safe across Container crashes.
- By default, emptyDir volumes are stored on whatever medium is backing the node – that might be disk or SSD or network storage.
- You can set the emptyDir.medium field to “Memory” to tell Kubernetes to mount a tmpfs (RAM-backed filesystem) for you instead.
- The location should of emptyDir should be in /var/lib/kubelet/pods/{podid}/volumes/kubernetes.io~empty-dir/ on the given node where your pod is running.


### Example of emptyDir – 1

```bash

apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### Example of emptyDir – 2

![image](https://github.com/user-attachments/assets/1e7a797c-5d41-4555-a757-9347b8184bc0)



### Example of emptyDir – 3

```bash

nano myVolumes-Pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myvolumes-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container-1
    
    command: ['sh', '-c', 'echo The Bench Container 1 is Running ; sleep 3600']
    
    volumeMounts:
    - mountPath: /demo1
      name: demo-volume

  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container-2
    
    command: ['sh', '-c', 'echo The Bench Container 2 is Running ; sleep 3600']
    
    volumeMounts:
    - mountPath: /demo2
      name: demo-volume

  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container-3
    
    command: ['sh', '-c', 'echo The Bench Container 3 is Running ; sleep 3600']
    
    volumeMounts:
    - mountPath: /demo3
      name: demo-volume

  volumes:
  - name: demo-volume
    emptyDir: {}

    ```
