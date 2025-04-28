# Kubernetes Basics: Understanding Manifest (YAML) Files

Welcome to the basics of Kubernetes manifests! 🚀\
This guide explains what a Kubernetes manifest is, its structure, and gives you easy examples to start writing your own YAML files.

---

## 📄 What is a Kubernetes Manifest (YAML File)?

A **manifest** is a simple `.yaml` file that tells Kubernetes:

- **What** you want to create (e.g., a Pod, Deployment, Service).
- **How** you want it to behave (e.g., how many replicas, what image to use).

**YAML** stands for **"YAML Ain't Markup Language"** — it's a human-readable way to describe data.

---

## 🧱 Basic Structure of a Kubernetes YAML File

Every manifest generally includes these fields:

```yaml
apiVersion: <API version>
kind: <Type of Kubernetes object>
metadata:
  name: <Name of the object>
spec:
  <Specification details for the object>
```

- **apiVersion**: Which version of Kubernetes API you are using (e.g., v1, apps/v1).
- **kind**: The type of object (Pod, Deployment, Service, etc).
- **metadata**: Name and labels to identify the object.
- **spec**: Configuration specific to that object.

---

## 🚀 Example 1: Simple Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

- **What it does**: Creates a single Pod running the **nginx** web server.

---

## 🚀 Example 2: Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

- **What it does**: Creates a **Deployment** that maintains **3 replicas** of the nginx pod.

---

## 🚀 Example 3: Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

- **What it does**: Creates a **Service** that exposes the nginx Deployment **inside the cluster**.

---

## 📚 Tips for Writing YAML Files

- Always use **spaces**, not tabs.
- Maintain **proper indentation** (2 spaces or 4 spaces).
- Double-check the **apiVersion** for the Kubernetes resource you are using.
- Store manifest files in version control system (VSC) - Github,ADO etc

---

## ✨ Useful Commands

| Action                 | Command                                                               |
| ---------------------- | --------------------------------------------------------------------- |
| Apply a YAML file      | `kubectl apply -f filename.yaml`                                      |
| Delete a resource      | `kubectl delete -f filename.yaml`                                     |
| View created resources | `kubectl get pods`, `kubectl get deployments`, `kubectl get services` |

---

In Kubernetes, **YAML manifests** are your way to **declare** what you want, and Kubernetes will work to make it happen.

---

