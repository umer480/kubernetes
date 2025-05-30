
# Health Probes:

### POD / Container Health Monitoring
### Application Health Monitoring


**Health Probes Types**

- Liveness Probe
- Readiness Probe
- Startup Probe

  
  ![image](https://github.com/user-attachments/assets/bcb40e4d-658a-4bfa-8c52-c6308d16802b)



### **Probe Test Methods**
**1**-  **HTTP**  -HTTP probes are probably the most common type of custom liveness probe. Kubernetes pings a path, and if it gets an HTTP response in the 200 or 300 range, it marks the app as healthy. Otherwise it is marked as unhealthy.

**2**-  **TCP** - Kubernetes tries to establish a TCP connection on the specified port. If it can establish a connection, the container is considered healthy; if it can’t it is considered unhealthy.

**3**-  **Command** - Kubernetes runs a command inside your container. If the command returns with exit code 0, then the container is marked as healthy. Otherwise, it is marked unhealthy.


# Liveness and Readiness Probe

In Kubernetes, `liveness` and `readiness` probes are used to monitor the health and availability of containers ( or application running inside the container).
While they may seem similar, they serve different purposes and have different effects on how Kubernetes manages pods.



`Kubelet monitors the liveness and readiness prob (Health Monitoring`



# ✅ Use Case Examples:

**1. Liveness Probe Use Case**


Let’s imagine another scenario where your app has a nasty case of deadlock, causing it to hang indefinitely and stop serving requests. Because the process continues to run, by default Kubernetes thinks that everything is fine and continues to send requests to the broken pod. By using a liveness probe, Kubernetes detects that the app is no longer serving requests and restarts the offending pod.


**Reference Diagram**

```bash
https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-setting-up-health-checks-with-readiness-and-liveness-probes
```



**another example**

Suppose you have a Node.js application that sometimes enters an infinite loop or deadlock. Kubernetes needs to restart the container if it stops responding.

```bash
livenessProbe:
  httpGet:
    path: /healthz
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5

```
**🧠 Explanation:**

- Kubernetes hits /healthz on port 3000 every 5 seconds.

- If the endpoint doesn't return a 200 OK, Kubernetes assumes the app is dead and restarts the container.


![image](https://github.com/user-attachments/assets/176dd644-64cf-4841-8b36-80a4c7060442)






![image](https://github.com/user-attachments/assets/219b0c94-41fc-42d0-8f2b-9498d8327596)



Probes can be configured in many ways. You can specify how often they should run, what the success and failure thresholds are, and how long to wait for responses. 


![image](https://github.com/user-attachments/assets/7a11225a-59f3-45a0-8a4e-c9bf01793f05)






**2. Readiness Probe Use Case:**


Let’s imagine that your app takes a minute to warm up and start. Your service won’t work until it is up and running, even though the process has started. You will also have issues if you want to scale up this deployment to have multiple copies. A new copy shouldn’t receive traffic until it is fully ready, but by default Kubernetes starts sending it traffic as soon as the process inside the container starts. By using a readiness probe, Kubernetes waits until the app is fully started before it allows the service to send traffic to the new copy.


### Reference Diagram :

Once app is ready to service the request then its added as an endpoint for a service ( then service routes traffic)


```bash

https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-setting-up-health-checks-with-readiness-and-liveness-probes


```


**another example**

Imagine a web app takes 30 seconds to initialize, load configs, or warm up before it can accept traffic. You don’t want it to receive requests before it’s ready.


```bash

readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5

```

**🧠 Explanation:**

- Kubernetes checks /ready every 5 seconds.

- If the probe fails, Kubernetes removes the pod from the Service endpoint list, and it won’t receive traffic.

- Once the endpoint returns 200 OK, the pod will start receiving traffic.


![image](https://github.com/user-attachments/assets/1b91b995-d35e-494f-9fdd-a2c8b4f43be4)


# **⚙️ Combining Both Probes**

```bash

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

```

**🧠 Effect:**

- Liveness keeps the container alive and restarts it when frozen.

- Readiness ensures it only receives traffic when it's ready to handle it.


![image](https://github.com/user-attachments/assets/7125a7d4-24e4-43ec-b28a-6ee37cb9c351)


![image](https://github.com/user-attachments/assets/2b0d8350-2bb8-45c1-88f0-5752a605e126)

![image](https://github.com/user-attachments/assets/b59419bb-71f2-4897-86f5-2e6fa25cbb65)






# LAB

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: readiness-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: readiness-demo
  template:
    metadata:
      labels:
        app: readiness-demo
    spec:
      containers:
        - name: app
          image: umerazeem/cloudlyncs:readinesstestimagev1
          imagePullPolicy: Always
          ports:
            - containerPort: 8080

# Liveness Probe - checks if the container is alive, restarts if fails:

          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10         # initialDelaySeconds: Wait before starting checks (to allow startup).
            periodSeconds: 15
            failureThreshold: 3             # failureThreshold:  Number of failures before action is taken.

# Readiness Probe - checks if the app is ready to serve traffic:

            readinessProbe:
              httpGet:
                path: /ready
                port: 8080
              initialDelaySeconds: 5
              periodSeconds: 10
              failureThreshold: 1


---
apiVersion: v1
kind: Service
metadata:
  name: readiness-service
spec:
  selector:
    app: readiness-demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer


```

**📝 Notes:**
You must configure the app to respond on /healthz and /ready with HTTP 200 when healthy/ready.




## Startup Probe:

`startupProbe` is important for containers that take a long time to start. It prevents Kubernetes from killing the container too early during its startup phase.

When you only use a livenessProbe, Kubernetes starts checking if the app is alive immediately. If your app takes, say, 60 seconds to start up—but your livenessProbe times out after 30 seconds—Kubernetes assumes the app is unhealthy and restarts it, over and over again.

🧨 This creates a loop where the app never gets a chance to fully start.

**When startupProbe is defined:** 

- Kubernetes uses only the startupProbe to check if the app has started.

- If startupProbe succeeds, then livenessProbe takes over.

- If it fails for too long (based on failureThreshold and periodSeconds), the container is considered failed to start, and it’s killed.


**Example:**

→ ` This gives the app up to 30 × 5 = 150 seconds to become healthy before Kubernetes kills it.`


```bash

startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 5

```

**🧠 When to Use startupProbe**
**Use it when:**

- Your app initializes slowly (e.g., compiles code, loads models, connects to DBs).

- You’ve experienced containers being restarted during startup.

- You want better separation of startup logic and liveness checks.





**Summary:**


![image](https://github.com/user-attachments/assets/c53ccfa7-73f7-4d18-9377-fcab74ea97cd)




![image](https://github.com/user-attachments/assets/347675fe-ef0b-4e0a-bcd0-ddf7798a43c6)


