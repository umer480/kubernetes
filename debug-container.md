# Debug Container / Emphemeral Container

When a pod is in `CrashLoopBackOff` and keeps restarting, you can't directly kubectl exec into it because the container never stays up long enough. However, Kubernetes provides a feature called ephemeral containers for exactly this situation.



![image](https://github.com/user-attachments/assets/43d82633-80f2-4260-8c72-5bba0e167947)



![image](https://github.com/user-attachments/assets/bc7d2950-cdee-4234-855b-c617cf2d3160)



## Sub-Topics:

1- Creating debug/ephemeral container
2- Debugging Examine - networking, connectivity. ( common network namespace, act as localhost)
3- Sharing Process namespace. Process Validation ( share process namespace, see process of main app)
4- Accessing and examining fileSystem of main container. ( /proc/$pid/root )
5- Alternative approach of troubleshooting without changing real/actual POD - Copy a pod, then troubleshoot.





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

**--target=<container-name>** — This makes the debug container share the process namespace of the crashing container, so you can inspect things like environment variables, filesystems, etc.




### Accessing a FileSystem of main contianer from the debug contianer:

It's even possible to access the file system of another container using the `/proc/$pid/root`

# See process id of main process :

```bash
ps -ef
ps aux
```

# change "8" to the PID of the Nginx process, if necessary

```bash
head /proc/8/root/etc/nginx/nginx.conf
```






## Key Point - Sharing Types within same POD:
**1-** Network namespace (they can talk to each other over localhost)  \
**2-** Process namespace (if configured using shareProcessNamespace: true)  \
**3-** Volume namespace ( all containers within the same pod can access same shared volumes)




### Kill / Restart process of a main conrainer:
You can signal processes in other containers. For example, send SIGHUP to nginx to restart the worker process. 

```bash
kill -HUP 8   # change "8" to match the PID of the nginx leader process, if necessary
ps ax
```

