
# Basic Structure of a YAML File:
How YAML files is logical structured in K8s

Supported Format of configuration files : **JSON** and **YAML**


✅✅✅ Kubernetes API Server only understands JSON as the wire format.✅✅✅

Every configuration file in k8s hsa 3 parts:

1- **Metadata** - of the object - Name and labels to identify the object.

2- **Specs** - configuration of the object

3- **Status** - current status of the object 

# **metadata:**

![image](https://github.com/user-attachments/assets/69ee44c4-acbc-4ae6-91e8-44b18b1598d2)



# **specification (specs):**

![image](https://github.com/user-attachments/assets/be0574e2-cc99-42d2-8da5-221d2a946a2c)



![image](https://github.com/user-attachments/assets/ec72da6c-2643-4514-b073-f9499eeffc48)


# **status:**

![image](https://github.com/user-attachments/assets/2e539451-4cda-49be-a380-73201178880b)

![image](https://github.com/user-attachments/assets/93a4de38-4ce5-4177-8213-10672ed69c0c)

![image](https://github.com/user-attachments/assets/b14bd64d-b69a-4263-a97f-dc475b7f766e)


# Where does k8s get this status data ?
etcd holds the current status of any k8s component

![image](https://github.com/user-attachments/assets/1d672092-cead-4b7a-8520-73440be81128)

**Example 1:**

```bash
apiVersion: v1                # Defines the version of the Kubernetes API being used
kind: Pod                     # Specifies the type of Kubernetes object (e.g., Pod, Service, Deployment)
metadata:                     # Metadata provides information about the object
  name: my-pod                # Name of the Pod (must be unique within the namespace)
  labels:                     # Key-value pairs for identifying and organizing objects
    app: my-app
spec:                         # Specification of the desired behavior of the object
  containers:                 # List of containers in this Pod
    - name: my-container      # Name of the container
      image: nginx:1.21       # Docker image to use
      ports:                  # Ports that are exposed
        - containerPort: 80   # Port number exposed by the container

```


**Example 2 :**

```bash

apiVersion: apps/v1                       # API version for Deployments
kind: Deployment                          # Defines the object type as a Deployment
metadata:
  name: my-deployment                     # Name of the Deployment
spec:
  replicas: 2                             # Number of Pod replicas to maintain
  selector:
    matchLabels:
      app: my-app                         # Identifies Pods belonging to this Deployment
  template:
    metadata:
      labels:
        app: my-app                       # Labels for the Pod template
    spec:
      containers:
        - name: my-container
          image: nginx:1.21
          ports:
            - containerPort: 80

```




# YAML -  Syntax Indentation

**✅  Validate Your YAML**
Before applying it, validate the YAML: to check for syntax errors.

```bash
kubectl apply --dry-run=client -f file.yaml 
kubectl apply --dry-run=client -f file.yaml -o json
```
Online tools like **yamllint** can also help.


**✅  Use Comments for Clarity**
Use **#** for comments.

This is especially useful for complex structures.



**✅  List Items with Hyphens**

Lists are defined using a hyphen (-) followed by a space.

Items should be indented properly under their parent.

```bash

spec:
  containers:
    - name: nginx
      image: nginx:1.21
    - name: redis
      image: redis:7.0

```

✅  Ensure Proper Mapping
Key-value pairs must have a : followed by a space.

**Avoid missing spaces after :.**

```bash

# Correct:
metadata:
  name: my-pod

# Incorrect:
metadata:
  name:my-pod   # 🚫 Missing space after `:`

```


**✅  Boolean Values Must be Lowercase**

Use true or false, not True or False.

YAML is case-sensitive.

```bash
spec:
  hostNetwork: true  # ✅ Correct
  restartPolicy: Always  # ✅ Correct
  restartPolicy: always  # 🚫 Incorrect

```


**✅  Avoid Trailing Spaces**
Trailing spaces at the end of lines can cause silent errors.
Trailing spaces at the end of lines in a YAML file are invisible, but they can cause parsing errors
Make sure lines end cleanly.


```bash

apiVersion: v1
kind: Pod   
metadata:    
  name: my-pod    # <-- There are invisible spaces here
  labels:         
    app: my-app   
spec:             
  containers:     
    - name: nginx    
      image: nginx:1.21    
      ports:         
        - containerPort: 80    
```

🔍 In the example above, the lines Pod, metadata, name, labels, app, spec, containers, name, image, ports, and containerPort have invisible trailing spaces. These are hard to spot but will break the YAML file.

✅ Example Without Trailing Spaces (Good YAML):

```bash
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: nginx
      image: nginx:1.21
      ports:
        - containerPort: 80

```



**✅  Quotes are Optional, But Sometimes Required**

If the value contains special characters (:, {}, []) or spaces, wrap it in double quotes.

**For example:**
```bash
"some text with spaces"
```


# Question !!!
Where we store k8s configurations files -  

store the config files with your applicaiton code - on VCS system  like GIT 
you can also have seperate own dedicated git repository for k8s config files




