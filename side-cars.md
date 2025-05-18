# Multi-Container POD / Side-Cars:

A Multi-Container Pod in Kubernetes is a single Pod that runs multiple containers that share the same network namespace, storage, and lifecycle.
These containers can communicate with each other using localhost and share volumes for data exchange.

![image](https://github.com/user-attachments/assets/42eb69d1-b5d7-4f17-a1d0-258a4abd8905)


**Real Example:**

**Main Container** → A Node.js application

**Sidecar → Fluentd** to stream logs to Elasticsearch/AWS CloudWatch/Azure Monitor etc


![image](https://github.com/user-attachments/assets/b65e7941-fac3-4953-b975-e85abd8100fd)


![image](https://github.com/user-attachments/assets/92164a27-5c4f-4e25-91d5-b83824e467f9)



**LAB:**

Sometimes, you might have two containers running inside the same pod. A simple example of this would be one container that generates web content continuously and another container that serves that content to users.
This setup is different from an Init Container. In the Init Container example, a "puller" container downloads static content only once, saves it to shared storage, and then exits. After that, the main web server container serves the downloaded content.
But in this new example, there is no "puller" that just runs once. Instead, there is a content generator that runs all the time as a sidecar container. It keeps creating new content and adding it to the shared storage, while the web server serves this fresh content as it is generated.


```bash

apiVersion: v1
kind: Pod
metadata:
  name: multi-container-lab
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      ports:
        - containerPort: 80
      volumeMounts:
        - name: web-content-dir
          mountPath: /usr/share/nginx/html

    - name: content-generator
      image: busybox
      command: 
        - sh
        - "-c"
        - |
          while true; do
            echo "Date and Time is $(date)" >> /web-content/index.html
            sleep 5
          done
      volumeMounts:
        - name: web-content-dir
          mountPath: /web-content

  volumes:
    - name: web-content-dir
      emptyDir: {}


```



# Access Specific Container"

```bash
kubectl exec -it multi-container-lab -c <container-name> -- /bin/sh
```

**#Testing :**
```bash
watch cat /web-content/index.html
```

