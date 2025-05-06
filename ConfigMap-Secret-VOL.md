# Use ConfigMap and Secret Volume/Mount/Properties file

But ConfigMap and Secret are also volume types - Using ConfigMap and Secret as volumes is actually a common requirement, e.g. think of applications that take configuration files as parameter on startup. Like prometheus, elastic search or your own java application taking a properties file or certificate file.



## How to reference as volume:

![image](https://github.com/user-attachments/assets/54f16ae1-5a46-4894-a5a9-9aafd8164d52)



![image](https://github.com/user-attachments/assets/c01c4dd0-a0e5-4193-b1a6-98c47524c992)




https://dev.to/techworld_with_nana/configmap-and-secret-as-kubernetes-volumes-3jna

# LAB:

```bash

apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-config-file
data:
  mosquitto.conf: |
    log_dest stdout
    log_type all
    log_timestamp true
    listener 9001
    
---
apiVersion: v1
kind: Secret
metadata:
  name: mosquitto-secret-file
type: Opaque
data:
  secret1.file:  c29tZXN1cGVyc2VjcmV0IGZpbGUgY29udGVudHMgbm9ib2R5IHNob3VsZCBzZWU=
  secret1.file:  c29tZXN1cGVyc2VjcmV0IGZpbGUgY29udGVudHMgbm9ib2R5IHNob3VsZCBzZWU=
  secret1.file:  c29tZXN1cGVyc2VjcmV0IGZpbGUgY29udGVudHMgbm9ib2R5IHNob3VsZCBzZWU=
    
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
        - name: mosquitto
          image: eclipse-mosquitto:1.6.2
          ports:
            - containerPort: 1883
          volumeMounts:
            - name: mosquitto-conf
              mountPath: /mosquitto/config
            - name: mosquitto-secret
              mountPath: /mosquitto/secret  
              readOnly: true
      volumes:
        - name: mosquitto-conf
          configMap:
            name: mosquitto-config-file
        - name: mosquitto-secret
          secret:
            secretName: mosquitto-secret-file    



```
### Create deployment without mappging configmap/secret

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
        - name: mosquitto
          image: eclipse-mosquitto:1.6.2
          ports:
            - containerPort: 1883

```


### Secret without bsae64 is possible?

Yes, you can use **StringData** instead of **Data** - kubernetes will aautomatically encode it in base64 and handle internally.


```bash

apiVersion: v1
kind: Secret
metadata:
  name: mosquitto-secret-file
type: Opaque
stringData:
  secret1.file: test123
  secret2.file: secretpass

```



### | (YAML block scalar)
| (YAML block scalar) is mandatory or highly recommended in YAML when you want to define multi-line literal strings — especially if newlines and formatting matter (e.g., for config files, private keys, or scripts).

✅ Example where | is necessary: Storing a TLS private key or certificate

```bash
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
stringData:
  tls.crt: |
    -----BEGIN CERTIFICATE-----
    MIIDdTCCAl2gAwIBAgIUOvJhDA==
    Aab9Nw==
    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN PRIVATE KEY-----
    MIIEvQIBADANBgkqhkiG9w0BAQEFAASC...
    qQKBgQC0MEcZ4h3+a8tr== 
    -----END PRIVATE KEY-----

```

🔍 Why | is needed here:
You must preserve newlines and indentation (especially for formats like PEM).

Without |, YAML will try to fold the string into a single line, breaking the content.


# ✅ Another example: Shell script in a ConfigMap or Secret

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: startup-script
data:
  start.sh: |
    #!/bin/bash
    echo "Starting app"
    /usr/bin/my-app --config /etc/config/app.yaml

```
❌ What happens if you don’t use | here :
YAML will flatten the string into one line:

```bash
**#!/bin/bash echo "Starting app" /usr/bin/my-app --config /etc/config/app.yaml**
```






### Creating SSL Certificates

**1- Using DATA :**

```bash
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCg==  # base64-encoded cert
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQo=  # base64-encoded key

```
**NOTE:**  data: field – requires base64-encoded content



**2- Using StringData**

```bash

stringData:
  tls.crt: |
    -----BEGIN CERTIFICATE-----
    MIIDdTCCAl2gAwIBAgIUOvJhDA==
    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN PRIVATE KEY-----
    MIIEvQIBADANBgkqhkiG9w0BAQEFAASC...
    -----END PRIVATE KEY-----

 ```

If you're writing your own YAML and want Kubernetes to encode it for you, you can use stringData: with multi-line literals (and this is where | is helpful):
Kubernetes converts this to base64 internally when you apply the secret.

🔁 So why use | in stringData?
**Because:**

PEM-encoded certs and keys are multi-line.

YAML requires | to preserve newlines and formatting.

Without it, your certificate will become a one-liner, breaking its structure and validity.



### How to update existing Secret or add new secret and apply to existing deployment ?

Kubernetes does not automatically detect changes to the secret data.

Mounted files are symlinked to a memory-backed volume and not updated in existing pods unless re-mounted (i.e., pod recreated).

kubectl delete pod -l app=<your-app-label> -n <namespace>   -->  Pods will be recreated by the Deployment with the new secret mounted.
