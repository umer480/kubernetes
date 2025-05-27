
# Health Probe
# POD / Container Health Monitoring 
# Liveness and Readness Probe

In Kubernetes, `liveness` and `readiness` probes are used to monitor the health and availability of containers.
While they may seem similar, they serve different purposes and have different effects on how Kubernetes manages pods.

Kubelet monitors the liveness and readiness prob (Health Monitoring)



# ✅ Use Case Examples:

**1. Liveness Probe Use Case**
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


**2. Readiness Probe Use Case:**

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






