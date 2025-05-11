# LAB - Services 

**Customer/Application Name :** alpha

# Two Tier Application:

**FrontEnd**    - phpMyAdmin ( UI/Web tool to manage MYSQL DB Server)
**Backend**     - MySQL DB instance ( Database)


- ClusterIP   - **internal Service**   -->  DB
- NodePort    - **External Service**   -->  WEB



# **Architecture/Diagram:**


![image](https://github.com/user-attachments/assets/e46458f9-9298-4e81-b4ea-9e93b5e030bd)


# Deploy Backend: (DB):

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpha-mysql-deployment
  labels:
    app: alpha-mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha-mysql
  template:
    metadata:
      labels:
        app: alpha-mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: Tech@12345678
          - name: MYSQL_DATABASE
            value: myappdb
          - name: MYSQL_USER
            value: umer
          - name: MYSQL_PASSWORD
            value: Tech@12345678
        ports:
          - containerPort: 3306

```



# Question !!! 
Why we deployed Backend/DB before FrontEnd ?   - Real Question -  keep in mind - Avoid unnecessery Troubleshooting and 


# Deploy Backend Service (ClusterIP - internal):

```bash
apiVersion: v1
kind: Service
metadata:
  name: alpha-mysql-service
  labels:
    app: alpha-mysql
spec:
  type: ClusterIP
  selector:                  # Matches Pods with the label **app: alpha-mysql**
    app: alpha-mysql
  ports:
    - port: 3306              # Service Port - Exposes port 3306 for internal access on ClusterIP.
      targetPort: 3306        # Routes to the Pod's container port 3306.

```



# Deploy FrontEnd: (WEB-UI):


```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpha-web-deployment
  labels:
    app: alpha-web
spec:
  replicas: 6
  selector:
    matchLabels:
      app: alpha-web
  template:
    metadata:
      labels:
        app: alpha-web
    spec:
      containers:
      - name: web
        image: phpmyadmin/phpmyadmin
        env:
          - name: PMA_HOST
            value: "alpha-mysql-service"  # Provide actual Service name of the MySQL instance
        ports:
          - containerPort: 80

```
# Deploy FrontEnd Service (NodePort - external):


```bash

apiVersion: v1
kind: Service
metadata:
  name: alpha-web-service
  labels:
    app: alpha-web
spec:
  type: NodePort
  selector:
    app: alpha-web
  sessionAffinity: ClientIP    # some applcaition need to enable it to manage cookies - if you have multiple replicas
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080



```

# Validation: Verify the Service and Endpoints:

**FE:**
```bash

kubectl get svc alpha-web-service
kubectl describe svc alpha-web-service
kubectl get endpoints alpha-web-service
```

**DB:**

```bash
kubectl get svc alpha-mysql-service
kubectl describe svc alpha-mysql-service
kubectl get endpoints alpha-mysql-service

```





**Further Validation :**

```bash

kubectl exec -it <mysql pod>  -- /bin/bash
mysql -u root -p 
show databases;
```

**#Login to WEB pod and see if you can connect to mysql service / DNS name.**

```bash

k exec -it <web-pod-name> -- /bin/bash
ping alpha-web-service   or ping alpha-web-service.svc.default.cluster.local
nslookup alpha-web-service.svc.default.cluster.local

```



**Access phpMyAdmin Externally:**

```bash
http://<Node-IP>:30080
```


# **Traffic FLOW:**


**User --> FE Service > Frontend POD >   Backend Service > Backend POD**




![image](https://github.com/user-attachments/assets/59072d91-2ed3-4a75-af01-10b4513394a8)




![image](https://github.com/user-attachments/assets/530ef0d1-d26f-47f2-a4bf-146a4daa681e)


