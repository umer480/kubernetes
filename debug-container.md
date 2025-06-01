# Debug Container / Emphemeral Container



When a pod is in `CrashLoopBackOff` and keeps restarting, you can't directly kubectl exec into it because the container never stays up long enough. However, Kubernetes provides a feature called ephemeral containers for exactly this situation.



1- Creating debug/empehmeral container
2- Debugging Examine - networking,conenctivity,
3- Sharing Process namespace. Process Validation
4- Accessing and examinging fileSystem of real container.
5-Copy a pod then troubleshoot.


When process namespace sharing is enabled, processes in a container are visible to all other containers in the same pod.



Syntax:
kubectl debug -it <target_pod_name> --image=<debug_image> --target=<container_name>   


kubectl debug -it nginx --image=busybox  -- bin/sh
kubectl debug -it nginx --image=busybox --target=nginx -- bin/sh



**🔍 What each part does:**
kubectl debug — Starts a new ephemeral container in the pod.

-it — Makes it interactive (like a terminal).

<pod-name> — The name of the pod that's crashing.

--image=busybox — The debug container image (you can use other images like ubuntu, debian, etc.).

--target=<container-name> — This makes the debug container share the process namespace of the crashing container, so you can inspect things like environment variables, filesystems, etc.





It's even possible to access the file system of another container using the /proc/$pid/root link.


# run this inside the "shell" container
# change "8" to the PID of the Nginx process, if necessary
head /proc/8/root/etc/nginx/nginx.conf




Network namespace (they can talk to each other over localhost)
Process namespace (if configured using shareProcessNamespace: true)


You can signal processes in other containers. For example, send SIGHUP to nginx to restart the worker process. 

# run this inside the "shell" container
kill -HUP 8   # change "8" to match the PID of the nginx leader process, if necessary
ps ax
