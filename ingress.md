# Ingress :
 ## Expose services securely - External Access - Outside Kubernetes Cluster 


`The primary purpose of an Ingress is to simplify the management of traffic routing and load balancing for services exposed to the internet or other external networks`

```bash

Ingress  -> incoming traffic
External Access --> Efficiently manage external access to services within a cluster, particularly for HTTP and HTTPS traffic

```
We prefer **ingress** over **NodePort** because it’s scalable, secure, and manageable, especially when dealing with multiple services or domains.

LB Service is limited : Layer 4, no advance routing decisions, no SSL Termination etc

# ingress benefits

- Centralized access control - Act as a centralized entrypoint for all external service
- Expose multiple services through a single external IP address, eliminating the need for a separate LoadBalancer or NodePort for each individual service
- provides User user-friendly hostname
- Advance/Smart Routing: host-based and Path/URL based routing (/api, /web, etc.)
- TLS/SSL termination in one place ( provides centralized cert management with HTTPS termination)
- Load balancing (Uses a single external IP)
- Cost savings (fewer LoadBalancers used in cloud)
- Security - Web Application Firewall (WAF) - Detect/Prevent OWASP Vulnerabilities

## Implementation of Centralized EntyPoint (ingress) 

## Ingress Controller: 
It requires an `Ingress Controller`, which is a specialized pod or plugin (e.g., NGINX, Traefik, AWS ALB Ingress Controller) that processes these rules and handles the actual traffic routing.
The **Ingress rules** you define in YAML are just configuration — but for them to work, you must deploy an Ingress Controller in your Kubernetes cluster. The Ingress Controller is the component that watches these rules and actually proxies the traffic to your services.

### Some Well-known ingress Controllers:
- Traefik
- Nginx
- HA Proxy
- Istio

```bash
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
```


### 🔹 Real Use Case Example

**Let’s say you have:**

```bash

myapp.com/frontend -→ Should go to your frontend service

myapp.com/api -→ Should go to your backend service

```


## LAB

# Deploy sample frontend+backend application:

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
          - "-text=Hello from backend"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5678


```


### Setting up ingress Controller:

# ✅ 1. Create a Namespace for Ingress


```bash


apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx


```

# ✅ 2. Deploy NGINX Ingress Controller and expose it:

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
labels:
  app: ingress-nginx   
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
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:latest
        args:
          - /nginx-ingress-controller
          - --configmap=$(POD_NAMESPACE)/nginx-configuration
          - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
          - --election-id=ingress-controller-leader
          - --ingress-class=nginx     # ← This tells the controller to process Ingresses with ingressClassName: nginx
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - containerPort: 80
        - containerPort: 443
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: NodePort
  selector:
    app: ingress-nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  - protocol: TCP
    port: 443
    targetPort: 443
    nodePort: 30443


```

# ✅ 3. Create ingress rules

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx      # ← This tells Kubernetes to use the "nginx" ingress controller
  rules:
  - http:
      paths:
      - path: /frontend
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
      - path: /backend
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 80




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
