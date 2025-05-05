### Ingress - Expose services securely 

We prefer **ingress** over **NodePort** because it’s scalable, secure, and manageable, especially when dealing with multiple services or domains.

# ingress benefits

- Centralized access control
- provides User Friendly hostname
- host-based and Path/URL based routing (/api, /web, etc.)
- TLS/SSL termination in one place ( provides centralized cert management with HTTPS termination)
- Load balancing (Uses a single external IP)
- Cost savings (fewer LoadBalancers used in cloud)


It requires an Ingress Controller, which is a specialized pod or plugin (e.g., NGINX, Traefik, AWS ALB Ingress Controller) that processes these rules and handles the actual traffic routing.
The **Ingress rules** you define in YAML are just configuration — but for them to work, you must deploy an Ingress Controller in your Kubernetes cluster. The Ingress Controller is the component that watches these rules and actually proxies the traffic to your services.



🔹 Real Use Case Example

**Let’s say you have:**

**myapp.com/api** → Should go to your backend service

**myapp.com/frontend **→ Should go to your frontend service

# **Using Ingress, you define routing like this:** ### ingress rules:

```bash
rules:
- host: myapp.com
  http:
    paths:
    - path: /api
      backend:
        service:
          name: backend
          port:
            number: 80
    - path: /frontend
      backend:
        service:
          name: frontend
          port:
            number: 80

```


### LAB ###

### Setting up ingress Controller:

# ✅ 1. Create a Namespace for Ingress


```bash


apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx

```

# ✅ 2. Deploy NGINX Ingress Controller (Simplified)

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-nginx
  template:
    metadata:
      labels:
        app: ingress-nginx
    spec:
      containers:
      - name: controller
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.47.0
        args:
          - /nginx-ingress-controller
          - --configmap=$(POD_NAMESPACE)/nginx-configuration
          - --watch-namespace=default
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - containerPort: 80
        - containerPort: 443

```

# ✅ 3. Expose the Ingress Controller via a Service (e.g., LoadBalancer or NodePort)

```bash

apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer  # Use NodePort if you're not on cloud
  selector:
    app: ingress-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
    - protocol: TCP
      port: 443
      targetPort: 443

```
# 🚀 Quick Test Tip
After applying both the controller and your Ingress rule, get the external IP:

```bash
kubectl get svc -n ingress-nginx
```

Then access your app via:

```bash
http://<EXTERNAL-IP>/api
```

# 🧠 How It All Connects:

- Your Ingress resources define rules (like /api → backend)
- The Ingress Controller watches for those rules
- When traffic hits your cluster via the controller's service IP (e.g., http://<external-ip>), the controller uses those rules to route to the correct service
