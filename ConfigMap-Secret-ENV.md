
# ConfigMap & Secret


### 1- Use as Environment Variable
### 2- Use as Volume/File


# What is ConfigMap:

A ConfigMap is a Kubernetes object used to store non-confidential configuration data in key-value pairs. It decouples environment-specific configuration from container images, making your apps more portable, flexible, and manageable.

🔧 **Real Use Cases: for ConfigMap:**
- Store app-level config like database URLs, app modes (dev, prod), or feature toggles, timeout etc
- Share environment variables with pods.
- Inject configuration files (e.g., .conf, .ini, .json) into containers.



# What is Secret:
Secret is just like ConfigMap but the difference is that it is use to store confidential/sensitive data. i.e password, connection string, private keys, access tokens, SSL certs



# LAB
🧪 Sample Use Case: Web + DB (2-tier app) using ConfigMap
 
 # Two Tier Application

-  web app that reads config from ConfigMap and Secret (like DB host,user,pass etc)
-  DB backend


![image](https://github.com/user-attachments/assets/d7c374f1-0a1a-4f93-8fcc-fbc23e223251)



1️⃣ ConfigMap YAML

```bash

apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  db_host: mongodb-service
 
---


apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ=

---


apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
              
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
        - name: ME_CONFIG_MONGODB_SERVER 
          valueFrom: 
            configMapKeyRef:
              name: mongodb-configmap
              key: db_host
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer  
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000

```


**Verification:** 
```bash

kubectl exec -it <mongo-express-pod-name> -- env | grep MONGO_INITDB_ROOT_USERNAME
kubectl exec -it mongo-express-64559f68f5-tprsn  -- env | grep ME_CONFIG_MONGODB_ADMINPASSWORD


k get configmap <configmapname>
k get configmap <configmapname> -o yaml
k describe configmap <configmapname>
k get secret <secretname>
k get configmap <csecretname> -o yaml
k describe secret <secretname>
```

Key Notes:
- ConfigMap and Secret must be exist before to use them in deployment.
- Same configmap/secret can be reference by many deployments/pods


![image](https://github.com/user-attachments/assets/fe3c81c7-a1d0-4d2b-97e8-6b19d11bfb75)




### How to encode/decode content:

```bash

encode string:
echo -n "Hello, World!" | base64

decode string:
echo -n "SGVsbG8sIFdvcmxkIQ==" | base64 --decode


encode file content:
base64 myfile.txt > myfile_encoded.txt

decode file content:
base64 --decode myfile_encoded.txt > myfile_decoded.txt

```


