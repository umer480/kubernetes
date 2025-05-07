### 🚀 ConfigMAP - Load all key-value pairs as an environment variable

The '**envFrom**' field loads all key-value pairs from env-config into the container as environment variables.


# create ConfigMap:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
data:
  APP_ENV: "production"
  DB_HOST: "mysql-service"
  DB_PORT: "3306"
```


# create pod
```bash


apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-config-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: env-config-app
  template:
    metadata:
      labels:
        app: env-config-app
    spec:
      containers:
        - name: app-container
          image: busybox
          command: ["sh", "-c", "env; sleep 3600"] # Prints environment variables and keeps the pod running
          envFrom:
            - configMapRef:
                name: env-config    # Reference to the ConfigMap


```

# VALIDATION:

The container will have these environment variables automatically set:
echo $APP_ENV    # Outputs: production
echo $DB_HOST    # Outputs: mysql-service
echo $DB_PORT    # Outputs: 3306





### 🚀 Secret - Load all key-value pairs as an environment variable

```bash

apiVersion: v1
kind: Secret
metadata:
  name: env-secret
type: Opaque
data:
  DB_USER: dXNlcm5hbWU=   # base64 encoded "username"
  DB_PASSWORD: cGFzc3dvcmQ=  # base64 encoded "password"
  DB_SERVER: cGFzc3dvcmQ=  # base64 encoded "password"
```

```bash


Deployment:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-env-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secret-env-app
  template:
    metadata:
      labels:
        app: secret-env-app
    spec:
      containers:
        - name: secret-env-container
          image: nginx
          envFrom:
            - secretRef:
                name: env-secret  # Reference to the Secret

```
