# Debug Container / Emphemeral Container

When a pod is in `CrashLoopBackOff` and keeps restarting, you can't directly kubectl exec into it because the container never stays up long enough. However, Kubernetes provides a feature called ephemeral containers for exactly this situation.


## UseCases:
-CrashLoopbackOff

-unexpected behavior of application

-networking issue

-tools/shell not available in main container



![image](https://github.com/user-attachments/assets/43d82633-80f2-4260-8c72-5bba0e167947)



![image](https://github.com/user-attachments/assets/bc7d2950-cdee-4234-855b-c617cf2d3160)



## Sub-Topics:

**1**- Creating debug/ephemeral container

**2**- Debugging Examine - networking, connectivity. ( common network namespace, act as localhost)

**3**- Sharing Process namespace. Process Validation ( share process namespace, see process of main app)

**4**- Accessing and examining fileSystem of main container. ( /proc/$pid/root )

**5**- Alternative approach of troubleshooting without changing real/actual POD - Copy a pod, then troubleshoot.





### Creating a debug/emphemeral Container:

**Syntax**:

```bash

kubectl debug -it <target_pod_name> --image=<debug_image>

Example:
kubectl debug -it nginx --image=busybox  -- bin/sh
```

### Sharing Process NameSpace:

`When process namespace sharing is enabled, processes in a container are visible to all other containers in the same pod.`

```bash

kubectl debug -it <target_pod_name> --image=<debug_image> --target=<container_name> 

Example:
kubectl debug -it nginx --image=busybox --target=nginx -- bin/sh
```


### 🔍 Explain Command Syntax:

**kubectl debug** — Starts a new ephemeral container in the pod.

**-it** — Makes it interactive (like a terminal).

**<pod-name>** — The name of the pod that's crashing.

**--image=busybox** — The debug container image (you can use other images like ubuntu, debian, etc.).

**--target=<container-name>** — This makes the debug container share the process namespace of the crashing container, so you can inspect things like process, filesystems, etc.




### Accessing a FileSystem of main contianer from the debug contianer:

It's even possible to access the file system of another container using the `/proc/$pid/root`

# See process id of main process :

```bash
ps -ef
ps aux
```

## Change "1" to the PID of the Nginx process, if necessary

```bash
head /proc/1/root/etc/nginx/nginx.conf
```





### Kill / Restart process of a main conrainer:
You can signal processes in other containers. For example, send SIGHUP to nginx to restart the worker process. 

```bash
kill -HUP 7   # change "7" to match the PID of the nginx leader process, if necessary
ps ax
```

**Kill the main process**:

```bash
kill -15 1
```




### Alternative way - How to share a process namespace 
You can also share process namespace with other contianers (of same pod) while deploying your application.

**Example**:

```bash

apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  shareProcessNamespace: true  # 🔍 This is required!
  containers:
  - name: nginx
    image: nginx


```

Note : in this case PID of main/init process will be other than `1`  - like `7`


## Debugging using a copy of the Pod

Sometimes Pod configuration options make it difficult to troubleshoot in certain situations. For example, you can't run kubectl exec to troubleshoot your container if your container image does not include a shell or if your application crashes on startup. In these situations you can use kubectl debug to create a copy of the Pod with configuration values changed to aid debugging.

**Adding a new container can be useful when your application is running** but not behaving as you expect and you'd like to add additional troubleshooting utilities to the Pod.

For example, maybe your application's container images are built on busybox but you need debugging utilities not included in busybox. You can simulate this scenario using kubectl run:

```bash
kubectl run myapp --image=busybox:1.28 --restart=Never -- sleep 1d
```


Run this command to create a copy of myapp named myapp-debug that adds a new Ubuntu container for debugging:

```bash
kubectl debug myapp -it --image=ubuntu --share-processes --copy-to=myapp-debug
```

![image](https://github.com/user-attachments/assets/7bb59153-4be3-43e0-8ab8-403e51c7b805)


## Copying a Pod while changing its command 

Sometimes it's useful to change the command for a container, for example to add a debugging flag or because the application is crashing.

To simulate a crashing application, use kubectl run to create a container that immediately exits:

```bash
kubectl run --image=busybox:1.28 myapp -- false
```


You can see using `kubectl describe pod myapp` that this container is crashing:

```bash
Containers:
  myapp:
    Image:         busybox
    ...
    Args:
      false
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
```

You can use kubectl debug to create a copy of this Pod with the command changed to an interactive shell:

```bash
kubectl debug myapp -it --copy-to=myapp-debug --container=myapp -- sh
```


```bash
If you don't see a command prompt, try pressing enter.
/ #
```

Now you have an interactive shell that you can use to perform tasks like checking filesystem paths or running the container command manually.


![image](https://github.com/user-attachments/assets/99550e66-4a4a-48cb-9ccb-0c5b892a41ba)



## Copying a Pod while changing container images

![image](https://github.com/user-attachments/assets/3500be4c-ec30-481d-a8a1-72f15d24794d)





# Key Point - Sharing Types within same POD:
**1-** Network communication  (they can talk to each other over localhost)  \
**2-** Process access (if configured using shareProcessNamespace: true)  \
**3-** Volume access ( all containers within the same pod can access same shared volumes)


